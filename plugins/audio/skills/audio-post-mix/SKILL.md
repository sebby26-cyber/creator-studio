---
name: audio-post-mix
description: "Assemble ordered narration with project-specific timing, then design music/SFX placement, levels, fades, crossfades and ducking against measured narration, applied as track automation in the editor (pre-mixed files are previews only). Holds the mix-level ceilings, the no-sudden-drops rule and the sparse-SFX rule, with their QC scans. Review audible continuity and record the actual edit."
---

# Audio post-mix

Generalized from an earlier standalone audio-post method. Minimum inputs: project audio/brand rules, selected speech files with explicit segment/line/chunk order, script/direction and production status. Brand levels and timing policy come from the external project, not universal defaults.

- **Takes:** ordered speech files, selected music/effect files tied to cues, and the project's levels and timing rules.
- **Makes:** a narration timing table, then the cue and level plan (placements, gains, fades, crossfades, ducking) applied in the editor, its level QC, and preview-only mixes for listening.
- **Tool:** the selected editor for the episode mix (every layer on its own track: `filmcraft` → "Assembly split"); FFmpeg (recipes in [references/ffmpeg-recipes.md](references/ffmpeg-recipes.md)) for voice prep, previews and QC, under `studio` → `references/local-jobs.md`. An audio-only program with no editor may be mixed with FFmpeg.
- **Save:** put the files in the production folder and add a line to `status.md`.

## Narration edit first
- Check completeness and ordering against render text, including every dialogue chunk. Listen across joins for cadence, breaths, changes of tone and missing/truncated words. Preserve breaths; do not overlap intelligible words to disguise seams.
- When the project permits post-production pauses, render/check stitched neighbors, measure native trailing/leading quiet and insert only `max(0, target pause - native quiet)`, with the project's pacing rule. Silence detection needs listening review; a breath is not disposable silence.
- With native timing (`v3_native_tags`), preserve rendered pauses: no digital-silence splicing or per-chunk normalization. Re-render a defective read when needed under the existing authorization.
- **Gain-matching chunks** (when the project asks for it, e.g. `voice.sleep.gain_match: chunk_median_lufs`): measure each chunk's integrated LUFS after any opener level dip (`audio-treatment` → "Opener level dip"), and gain each chunk to the median of the chunks, **except the final (outro) chunk**: it keeps its rendered level into the closing taper and is left out of the median. Log each chunk's measured level and applied gain (0 dB and "exempt: final chunk" for the outro). Then check the last join by ear and measure the level step across it; report a jump rather than re-matching the outro.
- Save a simple edit/timing table with source IDs/version, order, measured source/program in/out, native quiet and inserted gaps. For audio-only work this is the timing source for shots, captions and sound cues. For an episode with picture it is provisional: the pause pass changes the voice, and it is superseded by the timing table measured from the locked voice (`audio-treatment` → "Voice lock"; `studio` → "Episode order"), which is what shots, captions and cues are built from.

## Direction and sound design
Establish pacing and sound intent before generation; replace provisional timings with measurements after narration assembly. Write the cue list (cues, levels and notes) into the production folder. For an episode with picture this skill runs twice: the narration edit (join and timing table) before the pause pass and the voice lock, and the cue and level plan after the lock and `visual-director`'s spotting map, on the locked clock (`studio` → "Episode order").

Choose a continuous bed, evolving score, separate cues or silence according to the project's direction. Music may cross narration boundaries; do not restart it universally at each beat. Give each effect a narrative purpose and measured placement; no effects is valid (`sfx: []`). Silence before a reveal is a creative choice, not a fixed gap at every twist. Project `audio.mix_levels_db` supplies starting levels; check intelligibility and movement by ear.

**Levels: beds breathe, peaks never overpower.** Music and SFX can all be in the mix when it is engineered right. The aim is loudness that moves with the story and peaks that never overpower the voice. These ceilings apply unless the project sets its own (record which); the speech window itself is the project's or user's choice:
- **During speech** the music sits under the voice by the speech window the project or user has chosen (below). Until a choice is recorded in `status.md`, hold it at least 15 dB under on every momentary (400 ms) window, not only on average (a typical mean is about 18-21 LU under), and say in `status.md` that the window is still undecided.
  **Project or user decides: how strict the speech window is.** A bed held far under the voice leaves short pauses sounding like drop-outs, even when they are filled with room tone. The options: (a) 15 dB on every 400 ms window (the safest for intelligibility, the quietest bed); (b) 15 LU on the 3 s short-term plus at least 10 dB on every 400 ms window (a louder, steadier bed under speech, with less margin on single words). This skill does not choose; record the choice and who made it in `status.md`, and set the level table's `MUSIC-UNDER-SPEECH` flag to match.
- **In breathers and visual-only moments** the bed may rise, but its short-term (3 s) peak stays at least 3 LU below the voice's average short-term loudness.
- **SFX** momentary peaks stay at least 4 LU under the voice's average momentary loudness, and 6-10 dB under the voice where they overlap speech.
- **Dips land before the line.** A dip that lowers the bed for a line (a ceiling dip, a fall out of a breather lift, a duck) reaches its depth 0.10 s before the line's first word. Its ramp sits in the gap before the line, starts at least 0.10 s after the previous line, lasts at most 1.0 s and falls no faster than about 12 dB/s. A dip that is still falling after the line starts is heard as the voice cutting out. QC: no music momentary fall of 4 dB or more between 0.1 and 1.5 s after any line onset.
- **Variation:** across a long-form piece the bed's 10 s level spans at least about 8 dB: lower under dense or key lines, higher in breaks and transitions. Move it with 300-800 ms ramps, never a sudden drop. Tame swells and hits with keyframes or a gentle bus compressor or limiter.
- **Section breathers** (music-only breaks between story sections) are cut at assembly, before mastering: where they go, their lengths and the timeline mechanics are in `filmcraft` → "Section breathers". In the mix, the bed lifts into each breather within the ceiling above and settles back before the next line; it never stops dead, except the deliberate silence before a twist. An SFX tied to the moment may sit in the break.

**No sudden drops.** This is the one home of the rule. Music never drops suddenly to silence; the only exception is a planned pre-twist silence.
- Every music entry, exit and crossfade is a ramp. A crossfade between cues runs about 2 s, equal-power (both cues overlap on separate tracks with matching S-curves, or `acrossfade` with `c1=qsin:c2=qsin` in a preview), and sits inside a pause, not under a word.
- A dip under a key line goes to a low bed, never to silence. Entries and exits ride their ramps (300-800 ms for a level move, longer for a cue's first entry or last exit).
- **QC:** scan the music and SFX stems for drops greater than 6 dB within 200 ms ([recipe](references/ffmpeg-recipes.md#sudden-drop-scan-music-and-sfx-stems)) and classify each hit as a **designed fade** (on the plan, ramped), **in-source** (the same drop is in the source file at that source time) or an **edit** (a cut, a cue ending in a hole, a keyframe step). Edits must be 0 before export.

**Sparse SFX.** When an episode has only a few body SFX, each must clearly read: about 6-10 dB under the voice at its moment (not the 10-13 dB of a dense bed of effects), landing on its picture event, ideally inside a breather or rest so no word masks it. List their timestamps for the reviewer so each one is checked by ear.

Send selected music/effect briefs to `ai-music-and-sound`, `sound-render` or `suno`. Require cue-to-provider-result-to-durable-asset bindings and selected takes before mixing. Keep licensing checks and user review conditions.

## Apply the edit and mix
For an episode with picture, apply the measured offsets, gains, fades, crossfades and ducking in the editor, on each layer's own track (`filmcraft` → "Assembly split"); never hand the editor a single pre-mixed file as the episode audio. FFmpeg mixes are previews: name them `<name>_PREVIEW.<ext>`, label them preview-only in their sidecar and never use them as a source. An audio-only program may be mixed with FFmpeg. Inspect the actual command/filter graph or editor settings and review the rendered result; written instructions do not establish that an operation occurred. Save applied settings with exact inputs and output files.

[references/ffmpeg-recipes.md](references/ffmpeg-recipes.md) has tested commands for each operation. Know their limits before choosing one:
- **Join** only concatenates at a fixed rate and channel count, without inserted silence or normalization. Suitable for a native-timing join after boundary review. It does **not** implement a project's measured pause insertion; measure the native quiet and place each fragment directly, or use the selected editor, for that policy.
- **Dialogue stitch** adds a fixed gap on top of native quiet. Do not use it for target turn gaps; measure quiet and place each turn directly. Overlap only nonverbal material when direction calls for it.
- **Static mix** applies fixed gains and offsets with no automation. A fade needs an explicit start on program time (after the delay). For ducking, start-at-zero fades or moving beds, use the sidechain recipe, other explicit filters or editor automation. The mix ends at the voice duration unless you choose otherwise, so account for intended tails first.

## Review and next handoff
Review the whole program: speech joins/cadence, voice intelligibility, continuous music, effects on intended events, applied fades/ducking and the intended ending. Review visuals with the mix where applicable. Preserve human listening gates; report unreviewed audio honestly. Also check:
- **Level table:** a per-second voice vs music vs SFX momentary loudness table from the stems at their mix levels ([recipe](references/ffmpeg-recipes.md#per-second-loudness-table-voice-vs-music-vs-sfx)), with zero ceiling violations before export, and the bed's 10 s span.
- **Breathers:** in each section breather the bed is lifted, not dropped (silence only where a pre-twist silence was planned), and back at bed level before the next line.
- **Drops:** the sudden-drop scan on the music and SFX stems, every hit classified, zero edits.
- **Late dips:** the "Dips land before the line" check at every line onset.
- **Sparse SFX:** each body SFX listed with its timestamp and measured level against the voice.

For an episode built from timing tables in an editor, the level solver that turns these rules into keyframes (rides, lifts, falls, ceiling dips, the set-bounce-measure-trim loop) and the full conform QC checklist are in `episode-conform`; the numbers stay here.

The stems for these checks are bounced per track from the editor project (same start and length) into local scratch; they are measurement files, never sources.

For an audio-only program, `audio-master` then masters the mix to the project's target. For an episode with picture, loudness is reached on the editor's mix and measured on the export (`filmcraft` → "Assembly split"). Hand off the narration-edit/timing record, the cue and level plan, cue bindings with licences and credit lines, applied operations, review state and defects to final assembly. Final encoded output must be checked again for duration, truncation, sync, audible seams and loudness. A measured loudness pass cannot prove natural cadence or good sound design.
