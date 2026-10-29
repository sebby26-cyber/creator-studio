# Conform data contracts

Every layer of the episode is a table the generator reads. Values below are fictional examples; column order is the contract.

## Conventions
- **Format:** tab-separated text, UTF-8, with two header lines starting with `#`: the lock hash, then the column header (below). Text fields with spaces are quoted. Paths are absolute (the editor project references media by absolute path).
- **Lock hash:** the first `#` line of every table is `#voice_lock<TAB><sha256 of the locked voice file>`, the lock the table was built from; the second is the column header. The pre-flight (SKILL.md → "Pre-flight") stops the build on any mismatch. `timeline.json` carries it as `audio_clock.sha256`.
- **Time base:** program seconds (decimal, at least 3 places; 6 for anything that must hit a sample) is the master clock. Picture rows use **integer frames** at the project fps (frame `f` starts at `f / fps` s). Voice inserts use **samples** at the voice file's rate. Editor units are derived at build time: FilmCraft time is integer ticks, 254 016 000 000 per second, so one frame at 24 fps is 10 584 000 000 ticks (`filmcraft` → "Conventions that matter").
- **Two clocks:** `tables/source/` holds tables on the pre-breather clock (the locked voice's own times); `tables/program/` holds the same tables mapped through the breather ripple. The generator reads only `program/`; the mapping step writes it.
- **Levels:** dB, relative to the clip's base gain unless the column says absolute. Loudness readings are LUFS (M = momentary 400 ms, S = short-term 3 s).
- **Clock file:** `clock` holds the program length in frames, the outro join `t0` (frame and seconds) and the total breather length, all written by the mapping step.

## Picture placements (`place.tsv`)
| Column | Unit | Meaning |
|---|---|---|
| `shot_id` | text | id from the shot plan |
| `track` | text | picture track (main picture, overlay) |
| `in_f`, `out_f` | frames | program in (inclusive) and out (exclusive) |
| `path` | path | still or clip |
| `kind` | `kb` / `hold` / `clip` | Ken Burns still, frame hold of a clip, or a moving clip |
| `src_in_f` | frames | source in for a clip or hold (0 for stills) |
| `src_w`, `src_h` | px | source size |
| `z0`, `z1` | factor | zoom at in and at out, relative to the cover fit (1.00 = fills the canvas) |
| `ax`, `ay` | 0..1 | crop centre as a fraction of the free travel (0.5 = centred) |
| `mode` | text | `full`, `static`, or a named transition role (sting in, sting end) |
| `note` | text | why (variety swap, cut-in, hold) |

Example: `S014	V1	309	458	/…/stills/harbour-wide.png	kb	0	4096	2304	1.00	1.04	0.5	0.6	full	"slow push into the line"`

**Ken Burns keys** (one key at the first frame and one at the last frame of the clip): cover scale `s0 = max(W / src_w, H / src_h)` for canvas `W x H`; at each end `S = s0 × z`; visible window `ww = W / S`, `wh = H / S`; crop centre `cx = (src_w − ww) × ax + ww / 2`, `cy = (src_h − wh) × ay + wh / 2`; editor scale `100 × S` percent and position `(W/2 + (src_w/2 − cx) × S, H/2 + (src_h/2 − cy) × S)`. Shown width `ww` must respect the crop floor (`visual-director` → "Picture variety"). A crop rectangle given in pixels converts to `z = W / (rect width × s0)` and the centre fractions.

## Picture overrides (`pic_overrides.tsv`)
`shot_id, path|-, z0|-, z1|-, ax|-, ay|-, src_w|-, src_h|-, note` — `-` keeps the base value. Applied after the base table, so a variety swap or a move change is one row and the base stays diffable. An override never changes `in_f` or `out_f`; a timing change is a base-table change that goes through the ripple.

## Voice clips (`voice_clips.tsv`)
`src_start_s, src_end_s, program_start_s` — one row per voice clip on the voice track, all from the one locked voice file. Written by the mapping step from the breather table: segment `k` runs from breather cut `k−1` to cut `k` in source time and starts at its source start plus the summed breather lengths before it. Example: `62.458333	147.125000	64.958333`.

## Voice inserts (`inserts.tsv`)
`n, parent_sample, len_samples, type (comma|sentence|paragraph|transition), sentence_id, word_before` — every pause the polish pass inserted or set, in **parent-take samples**. Program time is derived, never stored: `lock_s = (parent_sample + Σ len_samples of earlier rows) / rate`, then mapped to program time through `voice_clips.tsv`. Example: `15	2190000	17280	comma	C3.s2	harbour`.

## Sentences and level classes (`sentences.tsv`)
`sentence_id, start_s, end_s, class (normal|slow|fast|key|fast_key), words_per_s, text` on the locked-voice clock; `class` drives the speech ride ([level-solver.md](level-solver.md)). Example: `C1.s4	27.810	29.030	slow	1.64	"nothing moved"`. A `silences.tsv` (`start_s, end_s, dur_s`, measured on the locked voice with the project's silence threshold) sits beside it.

## Breathers (`breathers.tsv`)
`n, cut_f (source clock), len_f, len_s, follows_sentence, kind (minor|section|turn|closing), last_words` — `cut_f` is a picture cut that already lies inside an existing pause. Example: `4	1499	60	2.500	C1.s11	section	"…and then it stopped"`. The mapping step adds `last_word_end_s, next_word_start_s` on the program clock for the solver and QC.

## Music cues (`music.tsv`)
| Column | Unit | Meaning |
|---|---|---|
| `cue_id` | text | |
| `track` | text | music track (active cues alternate between two tracks where they overlap) |
| `path` | path | |
| `src_in_s` | s | source in |
| `p0_s`, `p1_s` | s | program in and out |
| `role` | `active` / `alt` | an `alt` row is an alternate on a **muted** track |
| `slot` | text | story slot the cue serves (hook, build, turn, close); an alternate shares its active cue's slot, `p0_s`, `p1_s` |
| `anchor_src_s`, `anchor_prog_s` | s or `-` | a source moment that must land on a program moment (rippled with the program) |
| `base_gain_db` | dB | clip gain from calibration |
| `note` | text | |

Alternates carry the **same** keys, trims, bumps and routing as their active cue, so the user can swap by unmuting. An alternate whose anchor would need a negative source-in is clamped at 0 and the misalignment is logged.

## Music level keys
- `keys_<cue>.tsv`: `t_s, db` — the volume keyframes the solver writes (rides, lifts, falls, ceiling dips), relative to `base_gain_db`.
- `trims_<cue>.tsv`: `t_s, db` — slow trims on a separate gain effect (per window toward the bed target).
- `bumps.tsv`: `cue_prefix, a_s, t1_s, t2_s, b_s, db` — a ramped extra gain (`a`→`t1` up, hold, `t2`→`b` down) for a lift the volume key cannot reach (the editor's keyframe ceiling). Applied to every cue with that prefix, alternates included.
- `lifts.tsv`: `key, value` — per-breather lift peak (`B1`…), closer swell, refrain dip, and optional per-breather fall-end offsets in seconds.
- `ceiling_dips.tsv`: `iter, start_s, end_s, depth_db` — dips added by the ceiling loop; iterations accumulate, never rewritten.

## SFX cues (`sfx.tsv`)
`sfx_id, track, path, src_in_s, start_s, dur_s, gain_db, fade_in_s, fade_out_s|join, eq_preset, picture_event` — `start_s` on the program clock; `picture_event` names the shot or word it lands on. Example: `X04	A3	/…/sfx/door-slam.wav	0	211.420	3.0	-7.5	0.15	1.0	body	"S028 the door slams"`. Per-version changes go in `sfx_overrides.tsv` (`sfx_id, start_s|-, gain_db|-`).

## Markers (`markers.tsv`)
`time_s, name, comment, colour` on the program clock. One colour per kind, recorded in `status.md` (beat rests, breathers, crossfades and refrains, the turn, `t0`). The generator adds one marker per breather and one at `t0` itself.

## Captions
As-spoken SRT and VTT on the source clock, mapped with the same ripple (each timestamp through `ripple(t)`), never re-transcribed. Rules: no cue overlaps the next, no cue spans a breather, cue count unchanged by the mapping. When the voice and breathers are unchanged between versions, the captions are byte-identical to the parent's.
