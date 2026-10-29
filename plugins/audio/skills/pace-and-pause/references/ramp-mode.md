# Ramp mode: method and commands

Used only when the project sets `audio.pace.ramp: true` (see `SKILL.md` -> "Opt-in rule"). Without it, none of this runs and nothing slows down. Every number below is a project key or the skill fallback named in `SKILL.md`; no number here belongs to any one project.

Tools: whisper.cpp (`whisper-cli` plus a model file such as `ggml-small.en.bin`), ffmpeg/ffprobe, Rubber Band 3 or newer (for the R3 engine, `-3`). No other runtime. Checks and installs: `studio` → `references/dependencies.md`.

## 1. Split map: `rest_type` per boundary
The direction step writes one field per split-map entry:

| `rest_type` | Meaning | Target in ramp mode |
|---|---|---|
| `sentence` | sentence end inside a paragraph | `rest_ramp.sentence` at p |
| `paragraph` | paragraph break | `rest_ramp.paragraph` at p |
| `transition` | designed scene / transition rest | its own `target_s` (never snapped to the paragraph target); `rest_ramp.transition` at p only when `target_s` is `null` |

```json
{"id": "b07", "after_chunk": 7, "line": "...the last words.", "rest_type": "transition",
 "target_s": 4.0, "tag": null, "delivered_est_s": null, "owed_s": null}
```
An entry without `rest_type` is read as `paragraph` and flagged. Boundaries that are not splits are typed from the script: paragraph break = `paragraph`, else `sentence`.

## 2. Sentence alignment
```sh
whisper-cli -m <model.bin> -f <in>.wav -ojf -ml 1 -sow -of <name>_words
```
`<name>_words.json` holds one word per segment with `offsets.from` / `offsets.to` (ms). Script sentences end at `.`, `!`, `?`, `…` or `...` followed by whitespace or end of text (an ellipsis ends a sentence for alignment). Walk ASR words and script words in order (case and punctuation folded); a sentence whose words do not match cleanly is skipped and flagged. Sentence span = first word start -> last word end.

Silences:
```sh
ffmpeg -hide_banner -nostats -i <in>.wav -af silencedetect=n=<noise_db>dB:d=<min_s> -f null - 2> <name>_sil.txt
```
Boundary rest = the detected silence lying between sentence K's last word end and sentence K+1's first word start. None found: skip and flag (never cut into speech).

## 3. Targets
- Story start S0 = first word after any standing intro; E = last word end.
- p = (boundary time − S0) / (E − S0), bounded to 0..1.
- target = `start_s + (end_s − start_s) × p` (linear, `rest_ramp_curve`).
- `land` boundary: `max(target, land_rest_s)` if `land_rest_s` is set.
- Every target: `min(target, rest_max_s)`; a designed transition cut to the ceiling is flagged.
- wpm ceiling per sentence: mark ceiling, `min` with the `wpm_ceiling` line at p if set, then running minimum in order: `ceil[i] = min(ceil[i], ceil[i−1])`.
- Stretch factor: `min(max(1, measured / ceil[i]) × taper(p), max_stretch)`, `max_stretch ≤ 1.10`. Factor at the cap = `capped`.

## 4. Splice (rests filled per `room_tone_source`)
**Cut points** per sentence i, with `f = crossfade_ms` in seconds and `T` = the splice fade-out (0.100 s; `SKILL.md` → Rules):
- Segment start `a = onset − f`; segment end `b = silence_start + max(f, T)`, so the whole fade-out lies inside the measured silence and never starts on the word.
- Tail rule: `tail = silence_start − word_end`; if `tail < tail_min_s`, `b = max(b, word_end + tail_min_s)`.
- The fade-out starts at `silence_start` (on the stretched segment, at its stretched position) and runs to the segment end, so its length `L = b − silence_start` (stretched) is never under `T`.

**Steps**, one command each, run in order for every segment; replace each `<…>` with a number before running:
1. Cut (24-bit): `ffmpeg -hide_banner -nostats -n -i <in>.wav -af "atrim=start=<a>:end=<b>,asetpts=PTS-STARTPTS" -c:a pcm_s24le seg_<i>.wav`
2. Stretch, only if the factor is above 1.00: `rubberband -3 -t <factor> seg_<i>.wav seg_<i>_s.wav`
3. Read the stretched length: `ffprobe -v error -show_entries format=duration -of default=nw=1:nk=1 seg_<i>_s.wav`
4. Fades (fade-out linear in dB, about -24 dB 10 ms before the end, then to silence): `ffmpeg -hide_banner -nostats -n -i seg_<i>_s.wav -af "afade=t=in:d=0.010,afade=t=out:st=<len − L>:d=<(L − 0.010) × 4.17>:curve=exp,afade=t=out:st=<len − 0.010>:d=0.010" -c:a pcm_s24le seg_<i>_f.wav`
5. Rest fill, room tone from the take's own floor (default `from_program`; `SKILL.md` → fill rule): `ffmpeg -hide_banner -nostats -n -ss <floor_start> -t <floor_len> -i <in>.wav -af "aloop=loop=-1:size=<floor_samples>,atrim=duration=<rest_len>" -c:a pcm_s24le rest_<i>.wav`
6. Join, one crossfade per seam (segment into rest, rest into the next segment): `ffmpeg -hide_banner -nostats -n -i seg_<i>_f.wav -i rest_<i>.wav -filter_complex "[0:a][1:a]acrossfade=d=<f>" -c:a pcm_s24le join_<i>.wav`, then the same command with the joined file and the next segment, until the program is one file, `<name>_paced.wav`.

**Fade numbers.** `curve=exp` falls 100 dB over its duration, so `d = (L − 0.010) × 4.17` reaches about -24 dB 10 ms before the end; the last fade takes the final 10 ms to silence. [verified, FFmpeg 9.0.2, synthetic tone, L = 0.1] 5 ms RMS blocks fell in even steps of about 1.3 dB to -24 dB, then to silence at the end.

**Rest length.** Each crossfade overlaps its two inputs by `f`, and a rest has a crossfade at both ends, so the rest file is `rest_len = target − (quiet kept at the end of segment i) − (quiet kept at the start of segment i+1) + 2f`. It must be at least `2f`; if the kept quiet alone already exceeds the target, the rest is flagged as long, never cut into the kept tail. Get rate and layout with `ffprobe -v error -show_entries stream=sample_rate,channels -of default=nw=1 <in>.wav`. With a project room-tone file, cut the rest from it instead.

**Opt-in digital silence.** Only a rest **between** sentences, and only with `room_tone_source: "none"`, may be digital silence: make it with `ffmpeg -hide_banner -nostats -n -f lavfi -i anullsrc=r=<rate>:cl=<mono|stereo> -t <rest_s> -c:a pcm_s24le rest_<i>.wav` (here `rest_s` has no `+ 2f`: there is no crossfade) and join the faded segments and rests in order with the concat demuxer, `ffmpeg -hide_banner -nostats -n -f concat -safe 0 -i <name>_splice.txt -c:a pcm_s24le <name>_paced.wav`.

## 5. Measure gate commands
- Re-run section 2 on `<name>_paced.wav`; compare transcripts (G1).
- Each rest vs target: `|measured − target| / target ≤ rest_gate_pct/100` (G2); none above `rest_max_s` (G3).
- Whole program: `ffmpeg -hide_banner -nostats -i <file> -af ebur128=peak=true -f null - 2>&1 | sed -n '/Summary:/,$p'` (G6).
- C1 / C2: per-segment LUFS (`atrim` + `ebur128`) and wpm spread, reported as warnings only.

## 6. End tail
The closing taper takes `--end-word <new E>` and `--pad <end_tail_s>`. `end_tail_s` comes from the project; the skill fallback (20 s) is used only when the key is missing and is reported as such.
