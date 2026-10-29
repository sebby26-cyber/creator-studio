---
name: episode-conform
description: "Spec for conforming an edit plan into a finished episode in an NLE (FilmCraft first, written generically), built from timing tables so every placement, level key, marker and caption is exact and a rebuild is repeatable. Covers the table contracts (picture, voice clips and splits, music cues and muted alternates, SFX, breathers, level keys, markers, captions), one-pass generation with ripple, proving the generator against the previous version, the level solver, synchrony, voice-edit hygiene, picture variety, a QC checklist with pass thresholds and one-line measurements, and the producer review loop up to the user's gate. Spec only: the project builds and keeps its own conform toolkit in its tools/ folder."
---

# Episode conform

Turn an approved edit plan (`visual-director`'s `timeline.json`, `audio-post-mix`'s cue and level plan, the locked voice and its timing table) into the episode project in the editor and its export, **from tables, in one pass**. Hand-placed edits drift between versions; a table-driven build lands every layer on the same clock, can be diffed, and rebuilds identically.

This skill is a spec. **Nothing here is pre-built: the agent builds the conform toolkit in the project the first time this skill runs.** If the project already has one in its `tools/` folder (`<projects>/projects/<slug>/tools/`), use it; otherwise build one to this spec and save it there, never in the plugin. The plugins ship no scripts. Every tool built to this spec, and every QC scan, is proven on a planted defect and a clean input before it is trusted: `studio` → `references/local-jobs.md` → "Prove a check before trusting it".

- **Takes:** the locked voice and its timing table (sentences, words, inserts), the shot plan, the cue and level plan, the breather list, the selected picture, music and SFX files, the locked intro/outro pieces, the as-spoken captions, the project's canvas, fps, loudness target and mix ceilings, and the previous version (its project file, tables and export) when there is one.
- **Makes:** a new version folder holding the tables, the editor project, the export with its sidecar and hash, the captions, the QC evidence, the review notes and `status.md`.
- **Tool:** the selected editor driven headlessly (`filmcraft` over `filmcraft-cli` or MCP), the project's conform toolkit (`tools/`), and FFmpeg/ffprobe for measurement only, one job at a time under `studio` → `references/local-jobs.md`.
- **Save:** `productions/<id>/final/v<N>/` (layout under "Versions" below), and a `status.md` line per build. Never overwrite an earlier version.

## Where the rules live
This skill holds the conform method: the table contracts, the rebuild, the solver mechanics, the QC checklist and the review loop. Every rule it applies has one home elsewhere; it points there and never restates the numbers.

| Rule | Home |
|---|---|
| Track layout, the voice is placed and moved only, loudness reached on the editor's mix | `filmcraft` → "Assembly split" |
| Where breathers go and how long | `filmcraft` → "Section breathers" |
| Intro/outro join at `t0` | `filmcraft` → "Signature intro and outro" |
| Voice lock and retime | `audio-treatment` → "Voice lock" |
| Splice fades at every insert or split, word-tail decay, cut-off QC | `pace-and-pause` → "Splice fades at every insert or split" |
| Room-tone fill inside a sentence; digital silence only between sentences, as an opt-in | `pace-and-pause` → "Fill every insert inside a sentence with room tone" |
| Mix ceilings, bed lift and ramp lengths, dips land before the line, no sudden drops, sparse SFX | `audio-post-mix` → "Levels: beds breathe, peaks never overpower", "No sudden drops", "Sparse SFX" |
| Per-second level table and drop scan recipes | `audio-post-mix` → `references/ffmpeg-recipes.md` |
| Moves, cut-ins, dissolves, frozen holds, luma QC | `visual-director` → `references/treatments-and-moves.md` → "Transitions and holds" |
| Repeat limits, variation order, crop floors | `visual-director` → `references/treatments-and-moves.md` → "Picture variety" |
| Encoded-file checks (packet gaps, dropouts, padding vs drift) | `clip-assemble` → `references/ffmpeg-recipes.md` → "Check the final encoded file" |
| Loudness pass criteria | `audio-master` → "Pass criteria", with the project's target |
| Emotion tags fit the sentence type, tag QC before a paid render | `elevenlabs-voice-director` → 4a-7 |

## 0. Pre-flight
Run before any table is mapped or any build starts; a failure stops the build and is written to `status.md`.
- **Stale input.** Every table names the hash of the voice lock it was built from (data contracts → "Conventions"). Hash the locked voice (`shasum -a 256 <voice_locked>.wav`) and compare it with every table's lock hash. Any mismatch stops the build: that table was built against an earlier voice and is re-derived from the current lock through the clock map (`audio-treatment` → "Voice lock"), never patched row by row.
- **Source intake.** Probe every file the tables reference (the locked voice, music, SFX, intro and outro pieces, clips and stills) for sample rate, channels, fps, duration and size, and compare each with the project's rate (`audio.export.sample_rate`) and fps, and with the length the table plans to use from it. One file:
  `ffprobe -v error -show_entries stream=codec_type,sample_rate,channels,r_frame_rate,width,height:format=duration -of default=nw=1 <file>`
  A mismatch is either converted **once**, during prep, into a new file the tables then reference (never resampled again in the editor), or accepted and logged in `status.md` with the reason (for example a silent brand sting at a different fps that the editor conforms). A planned in/out beyond the file's measured length stops the build. Save the probe table under `evidence/`.

## 1. Data contracts
Every layer is a table. Column lists, units, the time base and examples: [references/data-contracts.md](references/data-contracts.md). In short:
- **One clock.** Program seconds is the master time base; picture rows are integer frames at the project fps; voice inserts are samples at the voice's rate. Editor units (FilmCraft ticks) are derived at build time, never stored.
- **Tables:** picture placements (shot id, asset path, in/out, crop, Ken Burns keys), voice clips and splits, voice inserts, sentences with level classes, music cues and their muted alternates, music trims and bumps, SFX cues, breathers, level keys (rides, lifts, falls, ceiling dips), markers, captions, and per-version overrides.
- **Two clocks never mix in one folder.** Tables on the pre-breather clock and tables mapped onto the version's program clock live in separate folders, named as such.

## 2. Rebuild method
1. **New version folder.** Copy the previous version's tables (never its exports) into `v<N>/`; the previous folder is read only from here on.
2. **Prove the generator first.** Before changing anything, rebuild the previous version from its own tables with the toolkit and diff the result against that version's saved project (the editor's sequence dump, e.g. `filmcraft-cli --project <p> inspect sequence`, normalized and diffed). Pass = no differences, or only rounding below one frame, each explained. A generator that cannot reproduce the last version cannot be trusted with the next one. Record the diff command and its result in `status.md`.
3. **Change only tables.** Every fix is a row change or an override table; the generator code changes only for a new capability, and then step 2 is repeated.
4. **Map the clock.** Breathers are applied as a ripple function over the source clock: `ripple(t)` = `t` + the summed length of every breather whose cut point is at or before `t` (a frame-exact version for frame tables). Apply it to every later layer together: picture, music in/out, SFX, markers, captions, level keys, sentences, silences and voice inserts. Nothing is moved by hand.
5. **Breathers.** A breather is a split of the voice clip at a picture cut that already sits inside an existing pause (`filmcraft` → "Section breathers" for where and how long). The voice becomes contiguous source segments with shifted program starts; each split gets the splice fade pair (`pace-and-pause`). The outgoing shot grows by the breather (a moving still keeps its move; a clip is never frozen); a music cue spanning the cut runs through; a cue anchored to a story point (a hit under the turn line, a source moment that must land at `t0`) has its source-in moved earlier by the same ripple so the auditioned moment still lands, clamped at 0 and logged when clamped.
6. **Generate in one pass.** Create the sequence, import only missing media (stop on any unresolved path), name and mute tracks, place every clip in one batch, collect the returned clip ids, then apply every effect, keyframe, gain, EQ, routing and marker in a second batch keyed by those ids. Count successes against the batch length; any failure stops the build. Snap music starts to the frame grid and shift source-in by the same amount. The build runs no FFmpeg.
7. **Dump and keep.** Save the sequence dump, the mapped tables and the batch files beside the project as evidence. A rebuild is minutes, so rebuild rather than patch.

## 3. Level solver
The solver turns the cue and level plan into editor keys on the music and SFX tracks; the voice is never touched. Mechanics, the editor gain stack, the set-bounce-measure-trim loop and editor limits: [references/level-solver.md](references/level-solver.md). The numbers it targets (ceilings, lift size, ramp lengths, dips landing before the line, SFX margins) live in `audio-post-mix`; the solver reads them, it does not define them.

## 4. Synchrony
Every layer lands on the story beat together; a table that moves one layer moves the others with it.
- **Music** hits sit under the key word (a swell, a bloom, a cue entry), and crossfades between cues sit inside rests or breathers, never under a word. Record each hit as a marker.
- **SFX** land on their picture event, inside a breather or rest where possible so no word masks them, or exactly on the beat they illustrate (`audio-post-mix` → "Sparse SFX").
- **Picture** keeps moving through a breather (the outgoing still's move continues, or the scene change happens inside the break), and a push builds *into* the cut that lands on the line, so the cut and the word arrive together.
- **Fixes preserve sync.** A level, variety or SFX fix never moves a cut or a line; if it must, it is a timing change and goes back through the ripple.

## 5. Voice-edit hygiene
Every inserted pause or split carries the splice fade pair, no word tail drops to digital silence faster than its natural decay, and every pause inside a sentence is filled with room tone: `pace-and-pause` → "Splice fades at every insert or split" and "Fill every insert inside a sentence with room tone". The conform adds two duties:
- **Map inserts onto the program clock correctly.** An insert's program time = its parent-take sample position plus the summed length of every earlier insert (the polish pass that added them shifted everything after each one), converted to seconds, then mapped through the voice clips' program starts (the breather ripple). A scan on parent-take times, or on times shifted only by breathers, measures the wrong place and can pass a defective voice. Validate the mapping before trusting the scan: every mapped insert must fall inside a measured silence of the post-edit voice.
- **A defect after the lock is a new voice version** (re-polish, re-lock, retime: `audio-treatment` → "Voice lock"), unless the user accepts a fade at the split in the editor; record which.

## 6. Picture variety
Apply `visual-director` → `references/treatments-and-moves.md` → "Picture variety" as an override table: per shot, a new source, crop region (pixels converted to zoom and centre) or move, never a new cut point or duration. Measure the repeat flags and each crop's shown width before and after, and run the picture checks on every changed shot.

## 7. QC
Run the checklist in [references/qc-checklist.md](references/qc-checklist.md) on the stems bounced from the editor project and on the export: per-second ceilings, drop scan, late dips, voice cut-off scan at every insert on the post-edit clock, digital-silence scan inside sentences and a listen at each insert, glitch scan, freeze, black and brightness pops, intro/outro null test, loudness and true peak, frame count, sync and drift, captions, and contact sheets of changed shots. Each check has its pass threshold and one measurement line; a failing check is a P1 or P2 item in the review loop.

## 8. Review loop
A producer-style review, triage of every reported defect to its layer and shape before any fix, a fix list ranked P1/P2/P3, a rebuild, and re-review until no P1 or P2 remains; then the user's gate. Format and rules: [references/review-loop.md](references/review-loop.md). The agent never approves; the export stays a draft until the user clears the gate.

## Other editors
The method is written FilmCraft-first, but it holds in any NLE (Descript, Premiere, Resolve and the like). Whatever the editor, these must hold, and the QC checklist proves them on stems bounced from that editor's project:
- One track per layer (`filmcraft` → "Assembly split"); never a pre-mixed file as the episode audio.
- The locked voice is only placed, moved and split at silences. No filler, silence or pause removal, no voice cleanup or enhancement (Descript's Studio Sound, an auto-level, a noise gate), no stretch or gain on it: `descript` → "Locked narrated episodes" for that editor's traps.
- Placements come from the tables, built in one pass and diffable against the previous version (section 2). Where the editor has no scripting or import format the toolkit can drive, place from the mapped tables by hand, then dump or export the timeline (an EDL or XML) and diff it against the tables before QC.
- The sequence runs at the project fps and sample rate; sources are checked first (section 0).
- Captions are mapped through the ripple, never re-transcribed by the editor.
- The export passes the same checklist as a FilmCraft export.

## Versions
- `final/v<N>/` holds: the export, its sidecar `<file>.json` and hash, captions (`.srt`, `.vtt`), the editor project, `status.md`, `tables/` (source clock and mapped clock in separate subfolders), `evidence/` (sequence dump, batch files, QC logs and tables, frame grabs), and `review/` (producer notes, fix lists, diagnoses).
- The project file references media by absolute path: source folders must not move between versions.
- The sidecar names the parent version and its hash. A muted alternate, a judgement call or an open user decision is listed in `status.md`.

## Pitfalls
- Changing a table before proving the generator reproduces the last version.
- A QC scan on the wrong clock (parent-take or pre-polish times): it measures the wrong place and reports a pass that was never tested.
- Splice fades of a few milliseconds at inserts where the words ran together: the tail is cut off.
- A well-faded insert filled with digital silence: against the bed it still plays as a skip. Checking the fade shape never finds it; check the fill.
- A ceiling dip that reaches depth after the line starts: heard as the voice cutting out.
- Muted alternates left without the active cue's automation, bus routing and anchor: a user's swap then sounds wrong.
- Judging glitches from the editor's live playback instead of the export (`clip-assemble` → "Playback stutter versus a file fault").
- Building on the user's open editor session instead of a copy.
