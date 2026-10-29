---
name: filmcraft
description: "Finish and cut video in FilmCraft, an open-source Premiere-style NLE in pure Rust, driven headlessly over its CLI (`filmcraft-cli`) or MCP server. Two jobs: (1) edit_sequence: clips with optional native stills (direct still-only workflow also documented) + the locked voice, music cues, SFX and brand intro/outro, each on its own track and mixed in the editor, to a graded, transitioned, captioned, loudness-checked final MP4; (2) cut_shorts: a final video + an approved cut list (picks timecoded against that video, from opus-clip / captions-and-clipping / descript) to one reframed, captioned short per pick. Local, zero credits. Use when the edit needs more than ffmpeg concat (transitions, Lumetri grade, titles, caption tracks, transcript cuts, interchange to Premiere/Resolve/Avid) or when approved clip picks should be cut automatically instead of by hand in CapCut/OpusClip."
---

# FilmCraft

FilmCraft (storytold/filmcraft, MIT OR Apache-2.0) is a non-linear editor: timeline, three-point editing, every trim
tool, Lumetri-style colour, ~93 video effects and ~30 transitions, caption tracks (SRT/VTT/SCC, burn-in), EBU R128
loudness meters, H.264/ProRes/DNxHR export, and FCP7 XML / FCPXML / OTIO / EDL / AAF / OMF interchange. Every action
is an engine command (650+), and the same commands are reachable from the CLI, a JSON-lines control channel and an
MCP server. This skill drives it **headlessly** on a `.fcproj` project file; nothing is billable.

Reference (copied from filmcraft `docs/` at commit 5231852, text unchanged apart from a header note):
`references/agents.md` (MCP tools, CLI, conventions), `references/control-protocol.md`, `references/captions.md`,
`references/transcripts.md`. Links to filmcraft files that were not copied point at that commit on GitHub.

## Setup (check first; install with the user's OK)

- This skill documents FilmCraft v0.4.0. Install that release (`filmcraft-cli-0.4.0-macos-universal.zip`, signed and
  notarized) from https://github.com/storytold/filmcraft/releases/tag/v0.4.0, or build that tag: `cargo build --release -p filmcraft-cli` (Rust >= 1.95;
  add `--features whisper` only if `transcript.generate` is wanted; weights download on request, never bundled).
  The v0.4.0 release builds are built without `whisper`. Newer releases may change commands: check
  `filmcraft-cli commands export --json` before relying on anything below.
  Show the user the exact release asset or build command and get an OK before installing (see `studio` → `references/dependencies.md`).
- Optional MCP: `claude mcp add filmcraft-headless -- /abs/path/filmcraft-cli mcp --project /abs/p.fcproj`.
  The CLI alone is enough for both operations below.
- Preflight: `filmcraft-cli commands export --json` must list `file.exportMedia`; if the binary is missing, install it as
  above (if the user declines, say so; do not fall back silently; `clip-assemble` is an alternative only if selected/authorized for edit_sequence).

## In Creator Studio
- **Takes:** for an edit, the selected clips and/or stills in shot order, the locked voice, the selected music cues
  and SFX, the brand intro/outro pieces, the cue and level plan from `audio-post-mix` and the measured timing table
  (plus finalized captions when required); for shorts, a final video and an approved cut list (the human has
  cleared its gates).
- **Makes:** a final MP4 (edit) or one reframed, captioned short per pick, plus the editable `.fcproj`.
- **Tool:** FilmCraft (`filmcraft-cli`, or its MCP server), headless and local; zero credits. If FilmCraft is missing,
  install it per `studio` → `references/dependencies.md` with the user's OK. `clip-assemble` is the FFmpeg alternative for a plain edit.
- **Save:** put each export and the `.fcproj` in the production folder, write each export's sidecar `<file>.json`
  (see `studio`) and add a line to `status.md`.

## Conventions that matter

- Time is integer ticks, 254 016 000 000 per second; every command also accepts `seconds`, `frame` or `timecode`.
- A new project must be created with `--save-as`: the first command is
  `filmcraft-cli --save-as p.fcproj exec file.newSequence ...`. `--project` only opens an existing `.fcproj` and
  fails with "No such file or directory" otherwise (the MCP `--project` form has the same rule: create the file first).
- After that: `filmcraft-cli --project p.fcproj --save exec <command> key=value ...` (values parsed as JSON when they
  can be); `run script.jsonl` runs many commands in order and stops at the first failure. Output is JSON, exit 1 on
  failure.
- Usage text is `filmcraft-cli help` (`--help` alone exits 2 with "missing subcommand").
- Inspect before and after: `inspect project` / `inspect sequence` give the ids every command needs.
- Look at the result: `file.exportFrame` (CLI) or `render_frame` (MCP) at shot boundaries, then open the PNGs.
- Edits are undoable (`edit.undo`); keep the `.fcproj` beside the outputs in the projects folder as the editable master.

## Operation `edit_sequence` (clips + optional stills + audio layers on their own tracks -> final video)

For an episode, build the project from timing tables in one pass rather than by hand: table contracts, the
rebuild-and-prove method, the level solver, the QC checklist and the review loop are in `episode-conform`. The
steps below are the FilmCraft commands that method drives.

### Pictures: clips, stills or both

Use clips, native stills, or a mix. Stills import directly (no conversion to clips). Keep source lineage, timing,
review state and QC evidence in `status.md`.

### Agent preflight

- **Draft vs. final.** A review draft may use selected, saved inputs that are not yet approved; label the export
  a draft (`pending_review` in its sidecar). A final requires the user's approval of its picture and audio inputs.
  Never record an approval on the user's behalf.
- Require at least one selected, saved picture (clip or still). Preview images or unselected substitutes cannot
  enter a final.
- Require a measured shot/timing record and the finalized narration-edit clock from
  `audio-post-mix`. A supplied shot list (e.g. from `narration-and-story-scripting`) can carry
  shot IDs, `start_s`, `duration_s`, `total_s` and `timing_source: "speech"`; verify those values against
  measured audio, not estimates. Keep source IDs/versions, picture bindings and source/program in/out in the
  working timing table. This record is required by agent preflight even though the shotlist port remains optional
  for handoffs using an equivalent external record; no new artifact type is needed.
- Require every audio layer as its own file for its own track ("Assembly split" below): the locked voice, the
  selected music cues and SFX (or intentional silence), the intro/outro brand pieces and the intended tail, plus
  `audio-post-mix`'s cue and level plan. A pre-mixed file is a preview, never the episode audio. Verify each
  layer's review state and measured duration. Project gain defaults or a nominal narration percentage do not prove
  intelligibility; the full mix, joins, fades and ending get a listening review on the export. Stop for missing
  components; for a final, also stop for missing approval.
- Read explicit project width, height, fps, program duration, caption requirements/style, integrated loudness target
  and peak ceiling (with units). Reconcile program duration with the full audio layout (voice, cues, brand pieces), including the tail;
  resolve any mismatch before editing. The finalized narration clock, mapped into program time including the
  intro offset, governs shot, caption and cue timing. Preserve already-rendered pauses; do not add beat gaps or
  double-count native quiet. The one exception is the planned section breathers (below), cut on the timeline. Timing
  changes require refreshed audio, shot record, cues and captions.
- When captions are required, require finalized **as-spoken**, time-aligned SRT/VTT and check it against the
  selected narration, including every spoken intro/outro. A draft script or transcript port alone is not proof of
  finalized captions. Record missing, unreviewed or unverified gates explicitly; registry readiness is not approval.

### Assembly split (every layer on its own track)

This is the one home of the assembly split; other skills point here. An episode is mixed in the editor, where every
layer can still be tweaked, not handed over as one pre-mixed file.
- **Tracks.** The `.fcproj` holds each layer separately: `V1` picture (`V2` for overlays and titles), `A1` the
  locked voice, `A2` music (plus a second music track, e.g. `A4`, where two cues overlap in a crossfade), `A3` SFX,
  the intro and outro brand pieces on their own picture and audio tracks, and captions on `C1`. Record the layout
  in `status.md` when the project names tracks differently.
- **FFmpeg's job is voice prep and QC measurement only:** the join, `pace-and-pause`, `audio-treatment` and level
  checks before the voice is locked, then measurement of stems and exports. It does not mix the episode.
- **Never hand the editor a single pre-mixed audio file as the episode audio.** The mix is built in the editor
  from `audio-post-mix`'s cue and level plan: clip gains, fades and crossfades, and track volume keyframes
  (`mixer.setKeyframe strip=A2 lane=volume time=<ticks> value=<v>`, or `mixer.writeAutomation strip=A2
  lane=volume points=[[<ticks>,<v>],...]` for a whole ramp; read the lane's current values with `mixer.inspect`
  first, since the CLI help does not state the lane's unit).
- **Previews.** A pre-mixed review preview (FFmpeg or a quick bounce) is fine for listening. Name it
  `<name>_PREVIEW.<ext>`, give it the sidecar `kind: "preview (not a source)"`, and never import it into a
  `.fcproj`, master it or use it as a source for anything.
- **The voice is locked** (`audio-treatment` → "Voice lock"). In the editor it is only placed, moved and split at silences (each split gets the splice fade pair in
  `pace-and-pause` → "Splice fades at every insert or split"): no gain,
  stretch, fade, normalization or re-processing. When voice prep changes the voice's length, the timeline (shots,
  cues, captions, markers) is retimed to the new voice, never the voice to the timeline.
- **Loudness** is reached on the editor's mix: export, measure the export (`audio-master` → its measuring steps),
  trim the `Mix` strip or track gains and re-export. Tame peaks in the mix (keyframes, a limiter insert on the
  `Mix` strip) rather than normalizing the exported file.
- **QC stems.** For `audio-post-mix`'s level checks, bounce each audio track solo from the `.fcproj` (same start,
  same length) into local scratch. They are measurement files: rebuildable, never sources.

Heavy exports and measurements follow `studio` → `references/local-jobs.md` (one job at a time, watchdog, local
scratch).

### Section breathers (music-only breaks between story sections)

This is the one home of the breather rule; other skills point here. A narrated explainer or documentary needs
short music-only breaks between story sections, or the story runs line into line for its whole length. The comma
and sentence rests inside the voice do not do this job. A breather is an **assembly** step: the editor spaces the
locked voice apart on the timeline. It is not a voice tag, a re-render or a voice-processing pass. Intra-line rests
belong to `pace-and-pause`; breathers are timeline gaps between sections, added after it.

**Where.** After the hook or opening question; at each beat or section change; after each big reveal line; before
the twist and after it; before the closing section. Aim for about 8-12 per 10 minutes, scaled to length (3-4 in a
2-3 minute piece). Never put two breathers back to back. Start from the pause notes' `breather` rows
(`elevenlabs-voice-director` → "Pause notes") and the visual plan's breather list, which read the same notes, so a
beat the direction planned lands in the picture and the music together; add or move one only with a note saying why.

**Length.** About 1.5 s for a minor beat, 2-3 s for a section change, about 3 s after the twist or turn (the
longest).

**Inside the break.**
- The music bed rises 3-5 dB into the break (or by the beat's spotted swell, when the spotting map puts one there;
  never two lifts stacked) and ramps back down over 300-800 ms, back at bed level before the next line. Never a hard stop to silence: the only exception is a deliberate 1-2 s silence before the twist. Bed levels
  and ceilings: `audio-post-mix` → "Levels: beds breathe, peaks never overpower".
- An SFX tied to that moment can sit in the break so it reads.
- The picture keeps its move (Ken Burns, clip motion) through the break, or uses the break for the scene change.
  Never a frozen frame and never black.

**Mechanics.** Breathers are cut in the episode's `.fcproj`, where every layer is on its own track ("Assembly
split": `A1` voice, `A2` music, `A3` SFX, with picture, markers and captions), from the narration timing table and
the spotting map. A pre-mixed file cannot take them: splitting it would cut the music.
1. List the breathers first: for each, the sentence it follows, its type and its length. Work from the **last
   breather back to the first**, so earlier cut points never move.
2. Split the voice clip at an existing silence after the line (`timeline.razor track=A1 time=<ticks>`), at the
   silence midpoint and never inside a breath or word. The voice audio itself is never altered: no gain, no
   stretch, no re-render; the only edit is the splice fade pair at the split (`pace-and-pause` → "Splice fades at
   every insert or split").
3. Slide every later voice clip later by the breather length, and ripple everything after the cut point by the same
   amount: picture on `V1`, SFX, music cues that start after it, markers
   (`markers.rippleSequenceMarkers on=true`) and captions (a sync-locked caption track follows insert edits;
   otherwise move each later caption). Use `timeline.move` with explicit clip ids and times; inspect the sequence
   after each breather. Captions shift; they are never re-transcribed.
4. A shot that spans the cut point is lengthened by the breather (its Ken Burns move retimed over the new
   duration, a clip held within the project's speed floor), or the next shot starts inside the break as the scene
   change. A music cue that spans the cut point keeps playing through the break.
5. Keep the music valid after the ripple: each cue still covers its span (check source length with `ffprobe`;
   loop or crossfade a cue that now runs short), crossfades still overlap, and no cue ends in a hole.
6. Shape the bed on the music track with track volume keyframes ("Assembly split" → `mixer.setKeyframe` /
   `mixer.writeAutomation`) or
   clip keyframes: up into the break, down before the next line.
7. Write a refreshed timing table: each voice clip's new program start (old start plus the sum of the breathers
   before it). The mix carries on in the same `.fcproj`; `audio-post-mix` reviews it from the QC stems. For an FFmpeg
   preview (`clip-assemble`), apply the same table to each voice fragment, cue, shot and caption.

**QC.**
- List each breather with the line it follows (sentence id and last words), its type and its length.
- Check that no stretch longer than about 90 s runs without a breather, outside the intro and outro.
- Check every breather for a moving picture (frame grabs at its start, middle and end: never frozen or black) and a
  lifted bed (measured music level up in the break, no drop to silence except the planned pre-twist silence).
- Check the voice is unchanged apart from start times (the voice stem with the breathers removed matches the locked
  narration), and that captions and markers still land on their words.

### Signature intro and outro

This is the one home of the intro/outro rule; other skills point here.
- **Locked brand pieces.** The intro and outro are made once, approved, and reused unchanged in every episode:
  picture, music, sonic logo and timing. Only the per-episode vocal changes (the teaser in the intro, and any
  per-episode outro line). Keep each piece's layers on their own tracks; never re-render or re-mix a brand piece for
  one episode. A change to one is a new version for the episodes that follow, recorded in the project.
- **Join rule.** `t0` is the end of the last spoken word of the episode body, measured from word times (not the end
  of the file or of the voice clip). The episode music fades out equal-power from `t0`, and the outro hangs off `t0`.
- **Outro order after `t0`:** a deliberate near-silence beat for emphasis, over a low ambience floor rather than
  digital zero; then the sonic logo; then the end-screen window; then a perceptual (eased) fade to black; then about
  0.5 s of true digital silence to end the file.
- **End-screen window.** On YouTube, end-screen elements can occupy the last 5-20 s of a video that is at least 25 s
  long. Keep that window clean (no captions or key text under the elements) and long enough for the elements
  planned. Other platforms: their own spec.
- **QC.** `t0` measured and logged; the near-silence beat measures above digital silence (the ambience floor is
  present); the sonic logo lands on its anchor; frame grabs across the end-screen window; the fade to black is
  smooth (the luma ramp check in `visual-director` → `references/treatments-and-moves.md`); the last ~0.5 s is
  digital zero.

### Assemble and finish

1. Create the project with `--save-as <project folder>/p.fcproj` and `file.newSequence` using project `name`,
   `width`, `height` and `fps`. Inspect the installed command descriptions for placement, picture-only controls,
   scaling and export parameters; use supported parameters rather than inferred flags.
2. Import the selected native stills and/or clips and every audio layer using absolute paths (`import`, or MCP
   `media_import` / `file.import`). Native still import does not require converting images to video clips.
   Place each layer on its own track ("Assembly split"): the locked voice on `A1` at its measured program start
   (`timeline.place item=<id> track=A1 seconds=<start>`) at unity gain, music cues on `A2` (and the second music
   track for overlaps), SFX on `A3`, the intro/outro brand pieces on theirs. Apply `audio-post-mix`'s cue and level
   plan as clip gains, fades and track keyframes. `loudnessLufs` is an export parameter (`file.exportMedia`), not an import one; never pass it (step 7).
3. Place picture on `V1` in measured shot order with explicit starts and durations. For stills, `duration` is integer
   ticks: `duration_s * 254016000000`; three seconds is `762048000000`. Example JSON-lines placement:
   `{"id":"timeline.place","params":{"item":4,"track":"V1","seconds":0,"duration":762048000000,"insert":false}}`.
   IDs and timing come from this project, not the example. For clips, use measured source in/out and picture-only
   placement: **exclude all source clip audio**. Naming `V1` alone is not proof of exclusion; inspect resulting
   audio tracks and remove any auto-placed source audio before export. Only the selected audio layers may feed the mix.
4. Fit each picture to the explicit canvas; do not assume import fills it. Inspect aspect ratio, crop and framing.
   The verified 1280x720 -> 2048x1152 fit used
   `effects.setParam clip=<timeline clip id> effect=motion param=scale value=160` (percent). Derive scale for
   other sizes/aspect ratios from the project's fit/crop policy and review frames. Apply transitions only where
   directed, with sufficient clip handles and no shortened program; dissolves use eased opacity and go through dark
   (`visual-director` → `references/treatments-and-moves.md` → "Transitions and holds"). Grade and titles follow
   project constants.
5. Import finalized captions with `captions.import path=<as-spoken .srt>` and `captions.setStyle` on the actual
   caption track (e.g. `C1`). Margin is normalized: the verified 140-pixel bottom margin on a 1152-pixel canvas
   was `140/1152 = 0.12152777777777778`; derive it for this project's dimensions. Check readability, safe area,
   wording and sync. The release build has no speech-to-text (`transcript.generate` requires `whisper`); use an
   existing finalized transcript/caption file. If transcription is needed and the user selects OpenRouter, follow `studio` → `references/openrouter.md` (`models.transcription` default/allowed/locked, route `openrouter`, price approval and credit caps). Review accuracy and provider timestamp support before converting to captions or using `transcript.set`; plain text is not timed words. Remove pauses/fillers only when requested and allowed, then repeat
   the timing/audio/caption handoff rather than shifting picture independently.
6. Inspect the sequence and actual audio-track placements. Timed picture coverage must be continuous from
   program zero through the full audio/program end, including intro, outro and tail; extend the intended final
   hold or directed outro when necessary. Do not end at the last narrated word or use a shortest-stream trim.
   Record the frame-aligned export end and expected frame count at project fps; any rounding must preserve the
   entire audio tail. Review frame grabs at every shot boundary, transitions, caption examples, each section breather and the final hold.
7. Export H.264 MP4 via `file.exportMedia` with explicit project `width`, `height`, `fps`, bitrate, `audio=true`,
   `burnCaptions=true` when required, and `wait=true`. Use the installed command's documented range/end controls
   to export the recorded full program duration. Do not substitute a generic 1080p preset or pass `loudnessLufs`.
   Audio export is documented in the CLI help (`filmcraft-cli` 0.4.0): `file.exportMedia` takes `audio` (default
   `true`), `range=entire|inOut|workArea|custom` with `startSeconds`/`endSeconds`, and the `export` subcommand leaves
   the sound out only with `--no-audio`. [unverified] No export with audio from a multi-track mix has been checked
   yet: confirm on the first one that every audio track reaches the file (step 8) before relying on it. Save the
   actual command and export log.
8. Measure the **final encoded file**, not just the timeline: ffprobe video/audio codecs (H.264/AAC),
   dimensions, fps, frame count and stream/program durations; measure decoded AAC integrated loudness and true
   peak using `audio-master` / an available audio meter. ffprobe metadata alone cannot measure loudness/peak.
   Compare unrounded measured AAC peak strictly against the project ceiling (no tolerance above it), and check
   integrated loudness against project policy. AAC may overshoot a compliant mix. If it fails, trim the `Mix` strip
   or tame the peaks in the mix ("Assembly split"), re-export and remeasure; do not normalize the exported file.
   Run the file checks in `clip-assemble` → `references/ffmpeg-recipes.md` → "Check the final encoded file": audio
   packet gaps and dropouts (a skip heard in the editor's live playback is checked on the export first), and the
   audio/video end difference (10-20 ms of encoder padding is not drift).
   Listen/watch the encoded whole program for intelligibility, seams, caption/audio sync, complete intro/outro and
   an untruncated tail. Keep measurements and listening/review state; an objective loudness pass cannot close
   unreviewed listening gates. Save it with its sidecar, naming all selected inputs and the `.fcproj`/timing/QC
   evidence in `status.md`, leaving required review gates open until actually cleared.
9. Optional handoff: `file.exportFcpxml` / `file.exportOtio` / `file.exportEdl` / `file.exportAaf` when a human will
   finish in Premiere, Resolve or Avid.

### Validation evidence and limits

A native CLI trial verified still import, explicit holds, scale fit, SRT import, caption margin and captioned H.264
export (picture only, no audio); see the project file for that evidence. Audio-enabled export and the steps above
remain unverified until a production run measures them.

## Operation `cut_shorts` (final video + approved cut list -> one short per pick)

Input is a cut list (an edit plan whose `source` and `clips[]` are required and whose
`start`/`end` are timecodes **in this source file**), from `opus-clip`,
`captions-and-clipping` or `descript`. Its `gates` must be clear (a human approved the picks); if not, stop and
list the open gates. A plain edit plan (a brief-level plan, or a CapCut paced edit) is not cut here.

1. New project (`--save-as`, see Conventions), import the source, `file.newSequence` at the plan's aspect ratio (9:16 -> 1080x1920), place the
   source on V1/A1. Reframe: the "Social Vertical 1080×1920" preset scales to fill (centre crop); when the subject is
   off-centre, set Motion position on the clip (`effects.setParam`) and check a frame grab.
2. Captions: if the plan asks for them, import the transcript-derived `.srt` (or `transcript.createCaptions` when a
   transcript has been loaded with `transcript.set`; this build cannot generate one) and style per the plan's `captions` spec.
3. For each `clips[]` entry: `export short-<id>.mp4 --preset "Social Vertical 1080×1920" --start <s> --end <e>`
   (or `file.exportMedia range=custom startSeconds=.. endSeconds=.. burnCaptions=true wait=true`). The hook is the
   first frame on screen: grab and look at it.
4. Save each short with its sidecar and a `status.md` line recording `source_start`, `source_end`, the cut-list
   clip id, `aspect_ratio` and whether captions are burned.

## Limits (from upstream's own honest assessment, 2026-10-05)

"Ready for real work" is ~50-60%: H.264 is the only delivery codec; Lumetri, keys and export render on the CPU
(slow on big footage); camera/phone files in the wild are less tested than conformance streams; speech-to-text is an
optional build feature, off by default. No automatic subject-tracking reframe. Check every export; a defect is fixed
and re-evidenced, never waived.

## Optional: publishing
The run ends at the finished files, each with its sidecar `<file>.json`, and `status.md`. After that, the user may
choose to publish: hand the file + sidecar to a separate publishing tool if wanted, or use WoopSocial or the platform's own tools. That choice is the
user's: offer it, never assume it, and take no publish action without the user's explicit OK for that post (what,
where, when). FilmCraft's Export mode lists destinations (YouTube, Vimeo, TikTok, Instagram, FTP), but upstream's
ROADMAP (M6, Export) still lists "publishing destinations" as missing at v0.4.0, so do not plan an upload through
FilmCraft; use its export presets (e.g. "YouTube 1080p Full HD", "Social Vertical 1080×1920") to make the file, then
the user's chosen publishing path.

## Creator Studio: structured output

- `edit_sequence` makes one final video (local file), with the `.fcproj` named beside it. `cut_shorts` makes one
  short per cut-list clip.
- Source: new creator-studio skill (2026-10-08) wrapping FilmCraft v0.4.0, commit 5231852, by the ArtCraft Team and
  the FilmCraft contributors, MIT OR Apache-2.0 (see `LICENSE-MIT`, `LICENSE-APACHE`, `NOTICE` in this directory).
  The ArtCraft name and logos are trademarks and are not used or vendored.
