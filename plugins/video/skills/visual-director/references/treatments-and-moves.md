# Treatments and moves

The ladder (cheapest that works), the move vocabulary, and tested local recipes for Ken Burns moves, grade matching and crops. Every command writes a new file (`-n`). Replace `<...>` placeholders and use the project's canvas and fps. Tested on FFmpeg 9.0.2 with a 4096x2304 still on a 2048x1152, 30 fps canvas [verified]: frame counts exact, moves inspected on first/last-frame grabs.

## Treatment ladder

| Rung | Treatment | Cost | Use when | Watch for |
|---|---|---|---|---|
| 1 | Reuse as is | free | QC passed, look matches, story state correct | baked text, before/after state, resolution |
| 2 | Ken Burns on a still | free | the line needs time on one image | push limit, repeated moves, drift into empty areas |
| 3 | Existing clip | free | real motion already exists and matches | look (grade it), what the motion claims, fps (conform by retiming, not frame blending), speed >= 0.6x |
| 4 | New still | paid (cheap) | no existing image shows what the line needs | locked image model only; brief first; `image-verify` |
| 5 | New motion (image-to-video, start/end morph) | paid (dear) | change over time, a transformation, a hero moment, or a hold that nothing else can carry | allowed video models only; a free fallback always planned |

## Move vocabulary

| Move | `treatment.move.type` | Typical scale | Reads as | Use on |
|---|---|---|---|---|
| Push-in | `push_in` | 1.00 -> 1.04-1.08 | attention, tension, "look closer" | establishing into the detail the line names; building passages |
| Pull-out | `pull_out` | 1.08 -> 1.00 | context, scale, release | reveals of size; settle and close passages |
| Pan / tilt | `pan_left`, `pan_right`, `tilt_up`, `tilt_down` | fixed 1.08-1.15 | travel along something | a band, a coastline, a skyline; keep screen direction consistent |
| Parallax-fake | `parallax_fake` | two crops of one still, different scales | depth without generation | a foreground subject against a far background; cut or dissolve between the two crops on a sentence end |
| Static hold | `static` | 1.00 or a fixed crop | weight, stillness | the turn, a reveal line, a hedge line; a few seconds at most (see "Transitions and holds") |

Rules: slow moves only unless the project says otherwise; one move per shot; never alternate push and pull shot after shot inside a section; the turn and reveal lines hold nearly static (a very slow drift, for example 1.00 -> 1.02), since a true zero move reads as frozen after a few seconds. Max push = `native short side / canvas short side` (a 2304 px still on a 1152 px canvas can take 2.0x; a 1600 px still only 1.39x) before upscaling.

## Ken Burns recipes (render each shot to a clip of its exact duration)

Values, written into each command as numbers (no shell variables): `<W>`, `<H>` the canvas (for example 1920 and 1080) and `<2W>`, `<2H>` twice it; `<fps>` the project fps; `<D>` the shot `duration_s`; `<F>` its frame count, `round(D × fps)` (6.59 s at 24 fps = 158 frames); `<F−1>` one less.

The still is first scaled to cover twice the canvas (smooth sub-pixel motion, centre crop to the canvas aspect), then `zoompan` makes one output frame per input frame (`d=1`) with `on` counting output frames.

**Push-in** (1.00 -> 1.06, centred):
```sh
ffmpeg -hide_banner -nostats -n -loop 1 -framerate <fps> -t <D> -i still.png -vf "scale=<2W>:<2H>:force_original_aspect_ratio=increase,crop=<2W>:<2H>,zoompan=z='1.00+0.06*on/<F−1>':x='iw/2-(iw/zoom/2)':y='ih/2-(ih/zoom/2)':d=1:s=<W>x<H>:fps=<fps>,setsar=1,format=yuv420p" -frames:v <F> -c:v libx264 -crf 18 -pix_fmt yuv420p shot.mp4
```

**Pull-out** (1.08 -> 1.00): `z='1.08-0.08*on/(<F−1>)'`, same `x`, `y`.

**Pan left to right** at a fixed 1.10: `z='1.10':x='(iw-iw/zoom)*on/(<F−1>)':y='(ih-ih/zoom)/2'`. Right to left: `x='(iw-iw/zoom)*(1-on/(<F−1>))'`. Tilts swap the roles of `x` and `y`.

**Off-centre anchor** (push toward a point at fractions `AX`, `AY` of the frame): `x='(iw-iw/zoom)*AX':y='(ih-ih/zoom)*AY'` with `AX`, `AY` in 0..1.

**Parallax-fake**: render two shots from the same still, one wide (`pull_out` or `static`), one cropped tighter on the foreground subject with a slow `push_in`, and cut between them on a sentence end.

Check: `ffprobe -v error -count_frames -show_entries stream=nb_read_frames -of default=nw=1:nk=1 shot.mp4` equals `F`. Look at the first and last frames:
```sh
ffmpeg -hide_banner -loglevel error -n -i shot.mp4 -vf "select='eq(n\,0)+eq(n\,<F−1>)',scale=640:-2,tile=2x1" -frames:v 1 shot-ends.png
```

## Transitions and holds
This is the one home of the dissolve, cut-in and hold rules; `filmcraft` and the plan point here.
- **Eased opacity, not linear.** [observed in FilmCraft; check other editors with the luma QC below] An editor that blends opacity in linear light makes a linear opacity ramp look uneven: brightness jumps at one end of the dissolve (a pop). Use perceptual, eased opacity curves (ease-in-out or S-curve keyframes); record the curve used. In FilmCraft (`filmcraft-cli` 0.4.0 command list) that is opacity keyframes on the clip, `effects.addKeyframe clip=<id> effect=<opacity effect> param=<opacity param> time=<ticks>` at each end with `effects.setParam ... value=<percent> time=<ticks>`, then `effects.setInterpolation clip=<id> effect=... param=... mediaTime=<ticks> interpolation=easeIn|easeOut|bezier|continuousBezier` on each keyframe (`effects.setKeyframe ... inInfluence=<0..1> outInfluence=<0..1>` shapes the S-curve). The effect and parameter names come from `filmcraft-cli inspect sequence` on the project; the help does not list them. A built-in transition is `sequence.applyVideoTransition clip=<id> effect=<name> frames=<n>` (`cross_dissolve` is the one the help names; `effects.list kind=VideoTransition` lists the rest, including any dip-to-black); a stock transition's curve is its own, so check it with the luma QC.
- **Dissolve through dark.** A dissolve between two different subjects goes through a dark dip (out to near-black, then in) and never double-exposes them. A straight dissolve is kept for two framings of the same subject or a deliberate morph-style fallback.
- **Clean picture after a key line.** Hold about 1 s of clean picture (no transition, no new text) after a key line before any transition starts.
- **Cut-ins.** A cut from one framing of a still to a tighter framing of the same still (a push-in by cut) needs at least a 1.45x scale change, or it reads as a glitch rather than a cut. Check that the tighter framing stays under the asset's max push.
- **No frozen holds.** A static hold (zero Ken Burns move) longer than a few seconds reads as a frozen frame. Give it a slow drift, or keep it short.
- **QC: luma ramp per transition.** For each dissolve or fade, measure per-frame mean luma across the transition's span and look for a pop (a single-frame step far larger than the rest). A reversal (luma falling, then rising) is expected once, at the bottom of a dissolve through dark; anywhere else it is a defect. Decode small and alone (`studio` → `references/local-jobs.md`); `-ss` is the transition start, so the window runs from 0 to its duration:
  ```sh
  ffmpeg -hide_banner -nostats -threads 4 -ss <transition start> -t <duration> -i final.mp4 -an -vf "scale=160:-2,signalstats,metadata=mode=print:key=lavfi.signalstats.YAVG:file=luma.txt" -f null -
  awk -F= '/YAVG=/ { if (n++) printf "%+.1f\n", $2 - p; p = $2 }' luma.txt   # frame-to-frame luma steps
  ```
  **Pass rule** (for a project's `tools/` luma checker, or by hand on the step list; no checker ships with the plugin): take the frame-to-frame steps of `YAVG` (8-bit luma, 0-255) inside the window; let `m` be the mean absolute step. A step whose absolute size exceeds `max(3 × m, 4)` is a **POP**. Report per frame: time, luma, signed step, POP flag; then a summary line with the window, frame count, mean step and pop count. Pass = 0 pops, and at most one sign reversal, at the bottom of a dissolve through dark. Pops are flagged by the rule; reversals are read off the signed steps.
  [verified, FFmpeg 9.0.2, synthetic clips] a 1 s fade from white to black showed even steps and 0 pops; a hard cut in the same window showed 1 pop. Pops must be 0 on every dissolve and on the final fade to black.

## Picture variety
This is the one home of the repeat limits and crop floors; the plan (Step 7 "Repetition budget") and `episode-conform` point here. Starting values from one long-form explainer [observed]; a project's own rules replace them.
- **Repeat flags.** Flag a source used in 3 or more slots, and a source shown again less than 60 s after its previous use ended. A designed cut-in (below) or a deliberate callback is not a repeat. Aim to clear the flags; a flagged reuse that stays must look clearly different (another region of the still and another move).
- **Vary from what is on disk first, in this order:** (1) unused or under-used assets, including a failed-QC take whose defect lies outside a crop rectangle (give the rectangle in pixels, then convert it to zoom and centre); (2) another crop or region of the same still with a new move (a push where it pulled before, an off-centre centre); (3) a frame pulled from a clip at an exact source time. New generation comes last, behind a live quote and the user's OK, with the prompt written by the separate prompt-writer (Step 8).
- **Crop resolution floor.** Shown width = source width ÷ crop zoom; it must stay at or above the canvas width (for a 1920-wide canvas, ≥ 1920 px) to stay sharp. A source at or under the canvas width (including a frame pulled from a canvas-sized clip) takes at most about 1.03x push.
- **Statics.** A designed beat static of 3-5 s is fine; a static of 6 s or more counts as a frozen hold ("No frozen holds" above) and gets a slow move (a zoom change of about 0.03-0.06 over the shot, same centre).
- **Swaps keep the timing.** A variety swap changes the source, crop or move only; every cut point and duration stays, so synchrony with the sound is unchanged. Re-run the picture checks after any swap.

## Look matching

Grades are the project's; these are the tools, not values. Apply one grade per off-look source and check it against an on-look reference frame side by side.

```sh
# warm -> cooler, less saturated (example values; derive from the project look)
ffmpeg -hide_banner -loglevel error -n -i in.png -vf "colortemperature=temperature=7500:mix=1,eq=saturation=0.85" graded.png
# crop out an off-look or off-claim region (keep the canvas aspect; check the push limit afterwards)
ffmpeg -hide_banner -loglevel error -n -i in.png -vf "crop=iw*0.8:ih*0.8:iw*0.1:ih*0.1" cropped.png
```

For clips, put the same filters in the clip's chain before normalization (`clip-assemble` recipes). Conform frame rate by retiming (`setpts`) or the assembler's conform, not by frame blending, and record the effective speed.

## When motion earns its cost

Write the reason on the shot (`brief.why_paid`):
- **Change over time** the line describes and a still cannot show.
- **Transformation**: one start/end morph shows a reveal (before-state to after-state) in one move. Its free fallback is a 1.5-2 s dissolve between two stills.
- **Hero moment**: hook image, turn, peak, closer.
- **Hold rescue**: a still would exceed the hold rule and no second framing or second asset works without spoiling a reveal.

Not reasons: "it would look nicer", filling a hedge line, motion for every shot. A paid clip shorter than its shot is slowed to no less than the project's min speed (default 0.6x) and then holds its last frame; buy a longer clip only when a live quote for that duration is in hand.
