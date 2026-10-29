---
name: audio-master
description: "Master program audio to the project's loudness target with ffmpeg two-pass EBU R128 (loudnorm), then measure the result independently (ebur128) and report pass/fail. Zero credits. Use for any final narration, podcast or video mix."
---

# Audio master

- **Takes:** the mixed audio (speech alone, or narration + music + effects) and the project's loudness target.
- **Makes:** the mastered audio file plus its measured loudness report (integrated LUFS, true peak, LRA, pass/fail).
- **Tool:** FFmpeg (`loudnorm` two-pass, then `ebur128`). Zero credits. Needs FFmpeg; see the `studio` skill's `references/dependencies.md` if it is missing.
- **Save:** put the master in the production folder and add a line to `status.md` with the report. For audio-only work this is the finished file (write its sidecar, see `studio`); video assembly must also check the encoded output. The user's listen gate follows.

**Episodes with picture.** An episode assembled in an editor is mixed there with every layer on its own track (`filmcraft` → "Assembly split"); this skill does not make a pre-mixed episode master for the editor. For those, use steps 1 and 3 to measure the editor's export (and the voice during voice prep), and reach the target by trimming the editor's mix. The full method below is for audio-only programs (podcast, narration-only delivery). Run the commands one at a time under `studio` → `references/local-jobs.md`, with `-map 0:a:0` or `-vn` when the input holds video.

Targets, sample rate, channels and bitrate come from the project (`audio.loudness`, `audio.export`), e.g. YouTube narration -14 LUFS / -1 dBTP; a sleep podcast -19 LUFS / -1.5 dBTP, 44.1 kHz / 128 kbps MP3. With no project target, ask; do not assume one.

## Method
1. **Pass 1, measure.** Print the analysis and read `input_i`, `input_tp`, `input_lra`, `input_thresh`, `target_offset`:
   ```sh
   ffmpeg -hide_banner -nostats -i in.wav -af loudnorm=I=-14:TP=-1:LRA=11:print_format=json -f null - 2>&1 | sed -n '/^{/,/^}/p'
   ```
2. **Pass 2, apply the measured values** into a lossless intermediate (`linear=true`, so a program that can reach the target by gain alone is only gain-shifted). If it cannot, FFmpeg falls back to dynamic mode; the report says which happened (`normalization_type`).
   ```sh
   ffmpeg -hide_banner -nostats -y -i in.wav \
     -af "loudnorm=I=-14:TP=-1:LRA=11:measured_I=<input_i>:measured_TP=<input_tp>:measured_LRA=<input_lra>:measured_thresh=<input_thresh>:offset=<target_offset>:linear=true:print_format=summary" \
     -ar 48000 -c:a pcm_f32le inter.wav
   ```
   A project closing fade goes in front of `loudnorm` in the same chain: `afade=t=out:st=<duration - fade>:d=<fade>,loudnorm=...`.
3. **Measure the intermediate independently** with `ebur128` (not loudnorm's own numbers):
   ```sh
   ffmpeg -hide_banner -nostats -i inter.wav -af ebur128=peak=true -f null - 2>&1 | sed -n '/Summary:/,$p'
   ```
   Read `I:` (LUFS), `LRA:` (LU) and `Peak:` (dBFS, used as true peak).
4. **Trim, then encode once.** If integrated loudness is more than 0.05 LU from target, close the gap with a pure gain trim (`volume=<delta>dB`), never letting the true peak go above the ceiling. Encode the final file from the intermediate:

   | Output | Codec flags |
   |---|---|
   | `.mp3` | `-c:a libmp3lame -b:a <bitrate, default 192k>` |
   | `.m4a` / `.aac` | `-c:a aac -b:a <bitrate, default 192k>` |
   | `.wav` | `-c:a pcm_s24le` |
   | `.flac` | `-c:a flac` |

   ```sh
   ffmpeg -hide_banner -nostats -y -i inter.wav -af volume=<delta>dB -ar 48000 <codec flags> out.mp3
   ```
5. **Re-measure the output** (step 3 on `out.*`). A lossy codec can shift loudness (an MP3 low-pass changed a -14.0 LUFS intermediate to -14.3 LUFS in testing). If the result is more than 0.1 LU off, adjust the trim and **re-encode from the intermediate**, not from the lossy file. Stop after three encodes and report what you have.
6. **Delete the intermediate** once the final file is measured.

## Pass criteria
`pass` is true only when integrated loudness is within +-0.5 LU of target **and** true peak is under the ceiling (allow 0.5 dB extra for lossy codecs, which can add inter-sample overs; say so in the report). Report the numbers: input and output LUFS, true peak, LRA, trim applied, `normalization_type`, number of encodes. Never claim a pass the measurements do not show.

## Rules
- Master once. No per-chunk normalization upstream unless the project says so.
- If a project treatment (`audio-treatment`) runs after mastering, do not re-master over it; its level change is intentional.
- Keep user listening gates and review cadence/seams separate from loudness. Record source mix/edit IDs, actual measurements, selected output and review state; a loudness pass is not creative approval.
