---
name: clip-assemble
description: "Assemble selected stills or generated clips over finalized audio using measured narration-edit timing, optional captions, and final encoded-output checks. Uses FFmpeg for review previews and single-layer files (one still under audio, an audio-only platform); a multi-layer episode is assembled in FilmCraft or the selected editor. Holds the final encoded-file checks (packet gaps, dropouts, end padding versus drift)."
---

# Clip assemble

- **Takes:** selected stills or clips in shot order, the finished audio (an audio-only master, or a `_PREVIEW` mix for a review preview) and its narration timing table, and captions/style when requested.
- **Makes:** the final video file, checked against the audio and timing record.
- **Tool:** FFmpeg (tested recipes in [references/ffmpeg-recipes.md](references/ffmpeg-recipes.md)), or `filmcraft` / the selected editor. Needs FFmpeg; see the `studio` skill's `references/dependencies.md` if it is missing.
- **Save:** put the video in the production folder, write its sidecar `<file>.json` (see `studio`) and add a line to `status.md`.

**Scope.** A narrated episode with voice, music and SFX is assembled in the editor with every layer on its own track (`filmcraft` → "Assembly split"); FFmpeg there only preps the voice and measures. Use this skill for review previews (named `<name>_PREVIEW.<ext>`, sidecar `kind: "preview (not a source)"`, never a source), and for single-layer deliverables such as one still under a finished audio-only program. Heavy commands follow `studio` → `references/local-jobs.md` (map only the streams used, seek before `-i`, one job at a time under the watchdog).

Minimum inputs: project/production identity, selected durable visual IDs/files in explicit shot order, finalized audio master and its narration-edit/timing record, captions/style when requested. Estimated script beats are provisional; resolve shot/caption/effect timings to the actual audio edit before export. Confirm visuals cover the complete intended program, including any tail.

Choose the assembly operation (commands are in the recipes file):
- **One still under audio:** the still-loop recipe (H.264/AAC, faststart). `-shortest` is compatible with a looping still; verify encoded audio and program duration afterward.
- **Stills slideshow:** use only after shot durations and order are resolved from the measured edit. `-shortest` can truncate audio when the visual sequence is shorter, so check coverage first and encoded duration and content afterward. Caption burn-in needs an FFmpeg build with libass; if it is missing, install it per the `studio` skill → `references/dependencies.md` with the user's OK.
- **Generated clips:** the clip-concat recipe normalizes size, frame rate and pixel format, then lays the master audio under the picture. Preserve the selected tool; explain unsupported or missing capabilities before changing it.

Section breathers (music-only breaks between story sections) follow `filmcraft` → "Section breathers", which holds the rule, mechanics and QC. For an FFmpeg preview, apply its refreshed timing table to the voice fragments, cues, shots and captions; never pad a pre-mixed file.

Record actual shot placements/durations, source audio-edit version and selected input IDs with the export. For FilmCraft, retain its editable project and inspect/export commands; do not remove narration pauses/fillers after timing is finalized unless that edit was requested, then refresh affected timing and captions.

QC: inspect frames at boundaries and with captions/labels; review the full audio/visual combination and retain user listening gates. Check the **final encoded output** for duration against the audio/edit record, missing/truncated content, caption/shot/effect sync, audible joins and project loudness/true-peak target (encoding may change peaks). [references/ffmpeg-recipes.md](references/ffmpeg-recipes.md) → "Check the final encoded file" is the one home of the file checks for any assembler (this skill or `filmcraft`): packet gaps and dropouts, and end padding versus drift. Report measurements and any unperformed review honestly; fix defects and recheck affected output.

Hand off exact output location, sidecar, hash, measured report, review state, outstanding defects and next action. Production ends at the reviewed final file; publishing is optional and separate (hand the file + sidecar to a separate publishing tool if wanted).
