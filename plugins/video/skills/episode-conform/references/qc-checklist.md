# Conform QC checklist

Run on the stems bounced from the editor project (each track solo, same start and length as the program, into local scratch) and on the export. Every command runs alone, under the watchdog, with `-ss`/`-t` before `-i` and only the streams it needs (`studio` → `references/local-jobs.md`). A check that fails is a P1 or P2 item in the review loop ([review-loop.md](review-loop.md)). Replace `<…>`; `t0` is the outro join, `P` the parent version's export.

A scan built for this checklist is proven first, including on a planted 0.35 s digital-silence hole inside a sentence (`studio` → `references/local-jobs.md` → "Prove a check before trusting it"). Save each check's raw output under `evidence/` and its verdict in `status.md`. A measurement pass is not a listening pass: the producer and the user still listen.

## Audio

- [ ] **Per-second ceilings.** Pass: 0 violation seconds; bed 10 s span as `audio-post-mix` requires. Method and flags: `audio-post-mix` → `references/ffmpeg-recipes.md` → "Per-second loudness table". Log per stem:
  `ffmpeg -hide_banner -nostats -i <stem>.wav -map 0:a:0 -af "ebur128=metadata=1,ametadata=mode=print:file=<stem>.r128.txt" -f null -`
- [ ] **Drop scan** (music and SFX stems). Pass: every drop classed designed fade / in-source / edit, 0 edits. Recipe: `audio-post-mix` → "Sudden-drop scan".
- [ ] **Late dips.** Pass: no music momentary fall of 4 dB or more between 0.1 and 1.5 s after any line onset (`audio-post-mix` → "Dips land before the line"). Read it off the music stem's r128 log at each sentence start from `sentences.tsv` (program clock).
- [ ] **Voice cut-off scan at every insert and split.** Pass per `pace-and-pause` → "Splice fades at every insert or split" (on 5 ms blocks the tail takes at least 35 ms to fall from −25 to −45 dBFS, and reaches ≤ −60 dBFS at the cut). **Use the post-edit time of each insert**: `lock_s = (parent_sample + Σ earlier len_samples) / rate`, mapped through `voice_clips.tsv` (data contracts → "Voice inserts"). Validate the mapping first: every mapped insert lies inside a measured silence of the voice stem. 5 ms blocks at 48 kHz:
  `ffmpeg -hide_banner -nostats -ss <t−0.200> -t 0.220 -i voice_stem.wav -map 0:a:0 -af "asetnsamples=240,astats=metadata=1:reset=1:measure_perchannel=none:measure_overall=RMS_level,ametadata=mode=print:key=lavfi.astats.Overall.RMS_level" -f null - 2>&1 | grep RMS_level`
- [ ] **No digital silence inside a sentence** (isolated voice stem). **Hard fail:** any run of digital silence of 50 ms or longer below −90 dBFS that lies inside a sentence (between its first word start and last word end, from `sentences.tsv` mapped to the program clock in `tables/program/`), per `pace-and-pause` → "Fill every insert inside a sentence with room tone". Runs between sentences pass only where the project opted in to digital silence. Scan on the post-edit clock:
  `ffmpeg -hide_banner -nostats -i voice_stem.wav -map 0:a:0 -af silencedetect=n=-90dB:d=0.05 -f null - 2>&1 | grep -E 'silence_(start|end)'`
- [ ] **Listen at each insert with the bed playing.** Pass: at every inserted pause inside a sentence, the producer hears the voice pause, not drop out, on the full mix (export or a mix bounce) on headphones. Log each insert time and the verdict.
- [ ] **Each insert sits at its punctuation.** Pass: for every insert inside a sentence (data contracts → "Voice inserts", post-edit time as in the cut-off scan), speech-to-text of about 2.5 s of the voice stem before the insert ends with `word_before` and about 2.5 s from the insert's end starts with the next script word, whole words on both sides (a fragment such as `ep-` ‖ `episode`, or `Cross` for `across`, means it is inside a word). Transcribe each side alone with a small English model (whisper.cpp `small.en`), never a large model alone: a large model filled a cut `a‖cross` back in as `across`. Split-point joins inside a sentence are checked the same way. Rule and method: `pace-and-pause` → "Punctuation pause placement". Save each pair of transcripts under `evidence/`. One side (whisper.cpp, run alone):
  `ffmpeg -hide_banner -nostats -ss <t−2.5> -t 2.5 -i voice_stem.wav -map 0:a:0 -ac 1 -ar 16000 -c:a pcm_s16le pre_<n>.wav && whisper-cli -m <model.bin> -f pre_<n>.wav -nt -np`
- [ ] **Voice unchanged.** Pass: the voice stem with breathers removed nulls against the locked voice, except within ±35 ms of any split with a splice fade. Null per segment (here one voice clip):
  `ffmpeg -hide_banner -nostats -ss <prog_start> -t <len> -i voice_stem.wav -ss <src_start> -t <len> -i voice_locked.wav -filter_complex "[1:a]volume=-1[n];[0:a][n]amix=inputs=2:normalize=0,astats=measure_perchannel=none:measure_overall=Peak_level" -f null - 2>&1 | grep 'Peak level'`
- [ ] **Glitch scan** (mix). Pass: every hit explained (voice, insert, bed or designed); 0 unexplained. 20 ms RMS blocks; a hit is a fall of more than 15 dB within 40 ms from above −50 dBFS:
  `ffmpeg -hide_banner -nostats -i export.mp4 -vn -map 0:a:0 -af "pan=mono|c0=0.5*c0+0.5*c1,asetnsamples=960,astats=metadata=1:reset=1:measure_perchannel=none:measure_overall=RMS_level,ametadata=mode=print:key=lavfi.astats.Overall.RMS_level:file=mix.rms20.txt" -f null -`
- [ ] **Breather levels.** Pass: each lift within ±0.5 dB of its target, measured post-bus ([level-solver.md](level-solver.md) → "Set, bounce, measure, trim"); bed back at level before the next line.
- [ ] **Sparse SFX.** Pass: each body SFX listed with its time and its margin under the voice per `audio-post-mix` → "Sparse SFX", landing on its picture event.
- [ ] **Intro/outro null test.** Pass: residual peak −inf (or at 24-bit rounding level) against the parent version for the intro (0 to its end) and the outro (from `t0` + the join fade to the end). Outro form, aligning each file's own `t0`:
  `ffmpeg -hide_banner -nostats -ss <t0_new+fade> -i export.mp4 -ss <t0_parent+fade> -i P.mp4 -filter_complex "[1:a]volume=-1[n];[0:a][n]amix=inputs=2:normalize=0:duration=shortest,astats=measure_perchannel=none:measure_overall=Peak_level" -vn -f null - 2>&1 | grep 'Peak level'`
- [ ] **Loudness and true peak.** Pass: `audio-master` → "Pass criteria" against the project's target and ceiling, measured on the decoded export; also read the mono fold-down when the project delivers to mono-capable devices:
  `ffmpeg -hide_banner -nostats -i export.mp4 -vn -map 0:a:0 -af ebur128=peak=true -f null - 2>&1 | sed -n '/Summary:/,$p'`
- [ ] **Encoded-file checks.** Pass: 0 audio packet gaps, every dropout a designed silence, end difference is padding not drift: `clip-assemble` → `references/ffmpeg-recipes.md` → "Check the final encoded file".
- [ ] **Tail.** Pass: the last ~0.5 s is digital zero (`filmcraft` → "Signature intro and outro"):
  `ffmpeg -hide_banner -nostats -sseof -0.5 -i export.mp4 -vn -map 0:a:0 -af astats=measure_perchannel=none:measure_overall=Peak_level -f null - 2>&1 | grep 'Peak level'`
- [ ] **Re-rendered voice only:** the tag QC passed before the render (`elevenlabs-voice-director` → 4a-7).

## Sync

- [ ] **Frame count.** Pass: equals the clock file's frame count.
  `ffprobe -v error -select_streams v:0 -count_packets -show_entries stream=nb_read_packets -of csv=p=0 export.mp4`
- [ ] **Sync and drift.** Pass: at an early, a middle and a late window, the voice in the export lines up with the locked voice at lag 0 (the residual is smallest at 0 within ±48 samples), with each window's voice-file offset = its program time minus the breathers before it. One lag test (repeat for lags around 0 by shifting `-ss` by one sample, `1/48000` s):
  `ffmpeg -hide_banner -nostats -ss <T> -t 10 -i export.mp4 -ss <T−breathers_before> -t 10 -i voice_locked.wav -filter_complex "[0:a]pan=mono|c0=0.5*c0+0.5*c1[a];[1:a]volume=<master_db>dB,volume=-1[b];[a][b]amix=inputs=2:normalize=0,astats=measure_perchannel=none:measure_overall=RMS_level" -vn -f null - 2>&1 | grep 'RMS level'`
  Late sync points (last word against its caption and shot cut, the sonic logo against its anchor) on frame grabs as well.

## Picture

- [ ] **Freezes.** Pass: only designed holds (a designed beat static is shorter than the frozen-hold limit in `visual-director` → "Picture variety").
  `ffmpeg -hide_banner -nostats -i export.mp4 -an -map 0:v:0 -vf "scale=480:-2,freezedetect=n=0.003:d=6" -f null - 2>&1 | grep freeze_`
- [ ] **Black.** Pass: only designed fades to black.
  `ffmpeg -hide_banner -nostats -i export.mp4 -an -map 0:v:0 -vf "scale=480:-2,blackdetect=d=0.04:pix_th=0.10" -f null - 2>&1 | grep black_`
- [ ] **Brightness pops.** Pass: 0 pops on every dissolve and fade (`visual-director` → "Transitions and holds" → luma ramp rule); at every changed cut, the largest luma step in ±3 frames is on the cut frame itself.
  `ffmpeg -hide_banner -nostats -ss <cut−0.125> -t 0.25 -i export.mp4 -an -map 0:v:0 -vf "scale=160:-2,signalstats,metadata=mode=print:key=lavfi.signalstats.YAVG" -f null - 2>&1 | grep YAVG`
- [ ] **Variety and crop floor.** Pass: repeat flags and shown widths per `visual-director` → "Picture variety", from `place.tsv` plus overrides (no media decode needed).
- [ ] **Contact sheets of changed shots.** Pass: the producer has looked at each changed shot at its start, middle and end (moving, sharp, on-look, no defect in frame). One shot:
  `ffmpeg -hide_banner -nostats -ss <shot_in> -t <shot_len> -i export.mp4 -an -map 0:v:0 -vf "fps=3/<shot_len>,scale=480:-2,tile=3x1" -frames:v 1 <shot_id>.png`
  A whole-film sheet for the review (one frame per 2 s): `-vf "fps=1/2,scale=240:-2,tile=8x6"` written as numbered PNGs.

## Captions

- [ ] **Captions.** Pass: cue count unchanged by the mapping, 0 overlaps, no cue spans a breather, cues land on their words (spot-check frame grabs); byte-identical to the parent when the voice and breathers did not change (`cmp parent.srt new.srt`). Overlap count:
  `awk -F' --> ' '/-->/{gsub(/,/,".");split($1,a,":");split($2,b,":");s=a[1]*3600+a[2]*60+a[3];if(s<e)n++;e=b[1]*3600+b[2]*60+b[3]} END{print n+0" overlaps"}' captions.srt`

## Delivery

- [ ] Streams, size, fps, codecs as the project requires (`clip-assemble` → "Check the final encoded file"); sidecar, hash and `status.md` regenerated; parent version untouched.
