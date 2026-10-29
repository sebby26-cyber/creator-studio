# Level solver

The solver writes the music and SFX automation from the tables. It never touches the voice. Every target number comes from `audio-post-mix` ("Levels: beds breathe, peaks never overpower", "Dips land before the line", "No sudden drops", "Sparse SFX") or from the project; this file is the mechanics. Values marked [observed] are starting points from one long-form explainer; the project's own values replace them and are recorded in `status.md`.

## Gain stack (per music clip, in order)
1. **Base gain** from calibration: the cue's measured source loudness against the voice, set so the bed starts at the project's distance under the voice. One value per cue (`music.tsv` `base_gain_db`).
2. **Volume keyframes** (`keys_<cue>.tsv`): the story curve (rides, pause rises, breather lifts and falls) plus ceiling dips.
3. **A separate gain effect** (`trims_<cue>.tsv`, `bumps.tsv`): slow window trims toward the bed target, and ramped bumps where a lift needs more than the volume keyframe can give.
4. **Music bus**: an optional gentle compressor on all music tracks (alternates included). Automate it so it never acts on a locked brand piece.
5. **Mix strip**: a limiter, then the master fader. Loudness is reached on this mix (`filmcraft` → "Assembly split").

## Speech rides (per sentence)
Each sentence's level is held flat across its words, set by its class in `sentences.tsv`, relative to the bed: a quieter bed under fast or key lines, a little more room under slow ones. [observed] normal −2.0 dB, slow −1.0, fast (≥ about 2.9 words/s) or key −3.0, fast and key −3.5; unlisted spans at the normal value. The class list (which lines are key) is the project's.

## Pause rises
Between two lines the bed may rise inside the gap and must be back before the next word. [observed] gap 1.0-1.8 s: +3.5 dB; ≥ 1.8 s: +4.5; a beat rest or ≥ 3 s: +5.0; measured from the louder neighbour, capped at the lower neighbour + 5 and at an absolute cap. The ramp up starts 0.10 s after the last word and the ramp down lands per "Dips land before the line"; each ramp lasts `clamp((gap − 0.2) / 2, 0.30, 0.70)` s, so it stays inside `audio-post-mix`'s ramp range. A gap under 1.0 s with a level change gets one linear ramp 0.10 s inside each edge; under 0.3 s, no move.

## Breather lifts and falls
Size and ceiling of the lift: `filmcraft` → "Section breathers" and `audio-post-mix`. Shape:
- **Rise** over about 0.8 s, starting just after the last word (0.10 s), then hold.
- **Fall** over about 1.2 s (1.0 s in a short break) that **reaches its depth before the next word, as `audio-post-mix` → "Dips land before the line" sets it**; a per-breather fall-end offset may end it earlier.
- **Special breathers** are rows, not code: a held suspense bed that falls to a hush just before the turn line (keep the hush short), a bloom that starts under a key word and peaks after it, a breather carried by a cue crossfade (equal-power, about 2 s, starting 0.10 s after the last word), a closing refrain dip and swell.
- **Curve shapes:** equal-power (cos/sin) for fades and crossfades, sampled at enough points (about 9) that the editor's interpolation does not step; the outro join fade per `filmcraft` → "Signature intro and outro".

## Rider
A slow rider evens the source's own swells: every 2 s, average the source's short-term loudness over ±2.5 s (measured once from the source file) and cut any excess over the cue's reference + 2 dB, by at most 6 dB.

## Ceiling dips (loop)
1. Bounce the stems, build the per-second level table (`audio-post-mix` → recipe) and list every violation.
2. Each violation at time `t` becomes a dip region `[t − 0.45, t + 0.05]` with depth = overshoot + 0.7 dB. Regions closer than 0.4 s merge at the deeper value, so the bed does not pump between words.
3. Within one iteration the deepest envelope wins; iterations add up (`ceiling_dips.tsv`, append only).
4. Ramps `depth × 0.12` s, clamped to 0.3-0.8 s (about 2.5 dB per 200 ms at most), placed so the dip is at depth before the word it protects (`audio-post-mix` → "Dips land before the line"). Floor: −18 dB below base unless the curve is already lower (a designed fade); hard floor −60.
5. Rebuild, re-measure, repeat until 0 violations. Log each iteration's count.

## Set, bounce, measure, trim (compressor allowance)
Keys computed on paper do not survive the bus compressor, the limiter and the codec. For every lift, swell and key level target:
1. **Set** the keys from the solver.
2. **Bounce** the music stem from the editor project, post-bus (so the compressor's loss is included), same start and length as the program.
3. **Measure** momentary loudness (100 ms steps) in each target window: from 0.5 s after the gap opens to 0.2 s before it closes; use the peak for a lift, the mean for a held hush.
4. **Trim:** `new_key = old_key + (target − measured)`. Pass within ±0.5 dB. Repeat from 2.
Then measure the export: a WAV bounce and the encoded file differ slightly (about 0.1 LU in one run) and the codec pads the end; the export is what passes.

## Editor gain limits
- **Volume keyframe ceiling** (FilmCraft: +15 dB). A key the trim loop pushes past it is clamped, and the remainder goes on the separate gain effect as a ramped bump (`bumps.tsv`, typically +1 to +2 dB), interpolated from the existing trim points so the base curve is unchanged outside the bump. The same applies when a clip's volume tops out at its maximum: raise its clip gain or add the bump, never the master.
- **Keyframes need both a sequence time and a media time**, and are clamped inside the clip (the last key sits just before the clip end).
- **A limiter's boost may do nothing**; the master gain is the fader after it. Moving the master changes the locked brand pieces too: check them with the null test (QC) after any master move, or trim the music instead.
- **Alternates** on muted tracks get every key, trim and bump of their active cue, through the same bus.
