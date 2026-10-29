# Heavy local jobs: memory, speech-to-text and disk

This is the one home of the resource rules for local processing (FFmpeg, speech-to-text such as `whisper-cli`, Rubber Band, NLE exports). Every skill that runs these tools points here. They exist because one careless command can freeze the machine or fill the disk mid-render.

## FFmpeg memory
- **Map what you use.** For audio work on a file that also holds video (an MP4, a MOV, an NLE export), always pass `-map 0:a:0` (or an explicit `-map "[label]"` for a filter graph's output) or `-vn`. A `-filter_complex` with no `-map` auto-selects the video stream too; the decoded frames buffer while the audio graph waits, at roughly 75 MB per second of start offset. One QC run grew to 17-22 GB this way and froze the machine.
- **Seek on the input.** Put `-ss` and `-t` **before** `-i` so FFmpeg seeks the input instead of decoding and discarding everything up to the start point.
- **One heavy job at a time**, with `-threads 4`. Never run two renders, or a render and a measurement pass on video, side by side.
- **Picture QC decodes small.** When a check needs frames (luma, frame grabs), scale down first in the chain (`scale=160:-2`) and drop the audio (`-an`).
- **Watchdog every heavy job** (the wrapper spec below): it kills the job when its resident memory (RSS) passes about 2 GB, and pauses it when free disk falls under the floor.

## Speech-to-text
Speech-to-text tools (`whisper-cli` and similar) take about 1.8 GB each with a medium or large model. Run one at a time, never alongside a render or an export, and serialize them with the same lock as FFmpeg (below), so a second transcription waits instead of starting.

## Disk
- **Work on local copies.** Copy only the current step's inputs into a local scratch folder (outside any cloud-synced folder: iCloud Drive, Dropbox, OneDrive, Google Drive and the like), run the step there, move the kept output to the production folder, then delete that step's scratch. Upload staging in a synced folder roughly doubles the disk cost of every intermediate, and sync can lock or rewrite files mid-job.
- **Intermediates stay out of synced folders.** Renders, conversions, stems bounced for QC and previews are written to local scratch; only kept outputs go where the project lives.
- **Check free disk before a big write** (a render, an export, a batch conversion) and pause below the floor (default 20 GB; the project may set its own). Report what would free space; never free it by deleting a source asset (`asset-persist` → "Cleanup: keepsake rule").

## The watchdog wrapper (spec)
Run every heavy job through a memory-and-disk watchdog. No script ships with this plugin, and a new project has none: the agent builds it on first use. If the project's `tools/` folder (`<projects>/projects/<slug>/tools/`, see `project-file.md`) already has one, use it; otherwise build one to this spec (a few dozen lines of POSIX `sh`, `ps` and `df`), save it there as `tools/heavy-job`, never in the plugin, and prove it with the acceptance check below before the first heavy job. Tested shape: macOS with FFmpeg 9.0.2.

**Interface:** `<wrapper> [-m MAX_RSS_MB] [-d MIN_FREE_GB] [-p DIR] -- command [args...]`. `-m` is the RSS cap in MB (default 2048), `-d` the free-disk floor in GB (default 20), `-p` the folder whose disk is checked (the scratch or output folder; default the current folder).

**Behaviour:**
1. Refuse to start when no command is given, when free disk on `-p` cannot be read (missing folder), or when free disk is already under the floor.
2. Start the job in the background and poll it once a second: its resident memory (`ps -o rss= -p <pid>`) and free disk (`df -Pk <dir>`).
3. RSS over the cap: send TERM, wait 2 s, send KILL, and exit 137.
4. Free disk under the floor: pause the job (STOP), print why, poll every 10 s until space is back over the floor, then resume (CONT). Never free space itself.
5. Stopping the wrapper (Ctrl-C, `kill`, hangup) resumes and terminates the job, so a killed wrapper never leaves an unwatched job running.
6. Otherwise wait for the job and pass its exit code through. Print every action on stderr, prefixed with the wrapper's name.

**Exit codes:**

| Code | Meaning |
|---|---|
| job's own | the job ran to the end (0 = success) |
| 64 | no command given |
| 66 | cannot read free disk for `-p` (missing folder): not started |
| 75 | free disk under the floor at start: not started |
| 137 | killed for going over the RSS cap |

**Acceptance check** before trusting a newly built wrapper: a normal job passes its exit code through; a job made to exceed a small `-m` is killed with 137; a floor above the available space refuses with 75; a missing `-p` folder refuses with 66.

Serialize with a lock around the wrapper, so the lock holder is the watched job (`tools/heavy-job` below is the wrapper built in the project, run from the project folder): `lockf -k` on macOS, `flock` on Linux. Use one lock file for all heavy jobs (FFmpeg, speech-to-text, Rubber Band, NLE export):

```sh
lockf -k "$TMPDIR/heavy-job.lock" tools/heavy-job -p "$SCRATCH" -- ffmpeg -hide_banner -nostats -threads 4 -ss 120 -t 30 -i episode.mp4 -map 0:a:0 -af ebur128 -f null -   # macOS
flock /tmp/heavy-job.lock tools/heavy-job -p "$SCRATCH" -- whisper-cli -m <model> -f voice.wav -ojf -of words   # Linux
```

- The cap watches the job's own process. A tool that forks workers (an NLE's export helper) needs its own limit or a lower `-m`; watch it in Activity Monitor or `top` on the first run.
- A killed job is a finding, not a retry: fix the command (usually a missing `-map` / `-vn`, or an `-ss` after `-i`) before running it again. Log the kill in `status.md`.

## Prove a check before trusting it
This is the one home of the rule; other skills point here. The plugins ship no checker scripts, so every check tool (a scan, a table builder, a timeline checker, a conform toolkit) is built by the model or the project from a spec. A tool built that way is not trusted until it has been proven:
1. **It fails on a planted defect** of the kind it hunts, made on synthetic or copied media in local scratch: a packet gap for the packet scan, a hard cut for the drop scan, a gap in the timeline for the timeline checker, and for any voice scan a hole of digital silence of about 0.35 s inside a sentence.
2. **It passes on a known-clean input** (a clean encode, a steady tone, the previous approved version).
3. **Both results go in `status.md`**: the tool, its version or hash, the planted defect, and what it reported on each input. A tool that cannot find its planted defect is a finding, not a pass.
4. **It runs on the right clock.** A scan of the voice uses the post-edit times of the inserts (`episode-conform` → "Voice-edit hygiene"); a pass on the wrong clock proves nothing.
5. **Heavy scans run in a subagent** with a bounded brief: the files, the timecodes or ranges, the thresholds and the tool in; a verdict table out (check | where | value | threshold | PASS/FAIL | evidence file). The subagent keeps to the rules above (one job at a time, watchdog, local scratch) and never changes a source.

Rebuilt tools are proven again; a proven tool kept in the project's `tools/` folder is re-proven when it or the media format changes.
