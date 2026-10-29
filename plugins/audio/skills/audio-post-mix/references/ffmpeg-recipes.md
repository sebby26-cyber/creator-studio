# FFmpeg recipes for audio post-mix

Tested with FFmpeg 6.1 (`amix` with `normalize=0` needs 4.4 or newer). Every command writes a new file; inputs are never overwritten. Run them one at a time under the watchdog in `studio` → `references/local-jobs.md`; on any input that also holds video, add `-map 0:a:0` or `-vn` (or map the filter graph's output), and put `-ss`/`-t` before `-i`. For an episode with picture, the mixes below are previews (`<name>_PREVIEW.<ext>`) or QC renders; the episode itself is mixed in the editor (`filmcraft` → "Assembly split"). Replace the `<...>` placeholders and run them directly. Always inspect the result by ear and with `ffprobe`; a command that ran does not prove the edit sounds right.

## Join fragments in order (native timing)
Converts each fragment once to a fixed rate and channel count and concatenates them with no inserted silence and no per-chunk normalization. Right for a native-timing join after boundary review. It does **not** insert measured pauses; for that, measure the native quiet at each seam and place each fragment directly (see "Place a clip at an offset").

```sh
ffmpeg -hide_banner -nostats -y -i f1.wav -i f2.wav -i f3.wav -filter_complex "[0:a]aresample=48000,aformat=channel_layouts=mono[a0];[1:a]aresample=48000,aformat=channel_layouts=mono[a1];[2:a]aresample=48000,aformat=channel_layouts=mono[a2];[a0][a1][a2]concat=n=3:v=0:a=1[out]" -map "[out]" -c:a pcm_s24le joined.wav
```
Use `channel_layouts=stereo` for stereo programs. Add one `[i:a]...[ai]` chain per fragment and raise `n`.

## Stitch dialogue turns with a fixed gap
`apad` adds the gap **on top of** each segment's native trailing quiet, so the real gap is longer than the number you set. Fine for a rough cut; for a target turn gap, measure the quiet and place turns directly.

```sh
ffmpeg -hide_banner -nostats -y -i t1.wav -i t2.wav -filter_complex "[0:a]aresample=48000,aformat=channel_layouts=mono,apad=pad_dur=0.3[s0];[1:a]aresample=48000,aformat=channel_layouts=mono,apad=pad_dur=0[s1];[s0][s1]concat=n=2:v=0:a=1[out]" -map "[out]" -c:a pcm_s24le dialogue.wav
```
The last segment gets `pad_dur=0` so the program does not end in silence.

## Mix music or effects under the voice (static levels)
The voice stays at 0 dB. Each bed has a gain in dB relative to the voice and an offset in milliseconds. A fade is optional; `st` is the fade start on **program time** (after the delay), so compute it from the offset.

```sh
ffmpeg -hide_banner -nostats -y -i voice.wav -i bed.wav -filter_complex "[0:a]aresample=48000[v];[1:a]aresample=48000,volume=-18dB,adelay=1000:all=1,afade=t=out:st=5:d=2[b1];[v][b1]amix=inputs=2:duration=first:normalize=0[out]" -map "[out]" -c:a pcm_s24le mix.wav
```
- `duration=first` ends the mix at the voice. If a tail after the voice is intended, use `duration=longest` and plan the voice padding.
- Add one `[i:a]...[bi]` chain per bed and list every `[bi]` in `amix`, raising `inputs`.
- Starting levels come from the project's `audio.mix_levels_db`; the ear decides.

## Duck a bed under the voice
Sidechain compression lowers the bed while the voice speaks. Tune `threshold`, `ratio`, `attack` and `release` by ear.

```sh
ffmpeg -hide_banner -nostats -y -i voice.wav -i bed.wav -filter_complex "[0:a]aresample=48000,asplit=2[v][vsc];[1:a]aresample=48000,volume=-12dB[b];[b][vsc]sidechaincompress=threshold=0.03:ratio=8:attack=20:release=400[bd];[v][bd]amix=inputs=2:duration=first:normalize=0[out]" -map "[out]" -c:a pcm_s24le duck.wav
```

## Place a clip at an offset (effects, cues)
```sh
ffmpeg -hide_banner -nostats -y -i sfx.wav -af "aresample=48000,adelay=<ms>:all=1" sfx_placed.wav
```
Then feed the placed file into the mix command as a bed. Offsets come from the narration timing table, not from script estimates.

## Measure silence at a seam
```sh
ffmpeg -hide_banner -i fragment.wav -af silencedetect=noise=-50dB:d=0.1 -f null - 2>&1 | grep silence_
```
Silence detection needs listening review; a breath is not disposable silence.

## Per-second loudness table (voice vs music vs SFX)
Checks the mix ceilings in `SKILL.md` ("Levels: beds breathe, peaks never overpower"). The spec below was tested with FFmpeg 9.0.2 and a macOS `awk` tool built to it, on synthetic stems. Inputs are the **stems at their mix levels and automation** (post-fader bounces of voice, music and SFX, same start and length); a raw source file says nothing about the mix. Write one EBU R128 log per stem (100 ms frames; `M` = momentary 400 ms, `S` = short-term 3 s):

```sh
ffmpeg -hide_banner -nostats -i <stem>.wav -map 0:a:0 -af "ebur128=metadata=1,ametadata=mode=print:file=<stem>.r128.txt" -f null -
```
Run it once per stem (`voice`, `music`, `sfx`), one at a time.

Then build the per-second table from the three logs (file order: voice, music, SFX). No table builder ships with this plugin: if the project's `tools/` folder has one, use it; otherwise build one to this spec (about 40 lines of `awk`) and save it there, never in the plugin, and prove it before use (`studio` → `references/local-jobs.md` → "Prove a check before trusting it").

**Parsing.** Each log has a `pts_time:<t>` line per frame followed by `lavfi.r128.M=`, `lavfi.r128.S=` and `lavfi.r128.I=` lines. Read `M` and `S` per frame per stem; take the voice's integrated loudness `VI` from its last `I` value.

**Speech gate.** `gate = VI - rel` (default `rel` = 10 LU). A frame is a speech frame when the voice `M` is at or above the gate. Voice average `aM` = mean voice `M` over speech frames; voice average `aS` = mean voice `S` over frames whose voice `S` is at or above the gate. No speech frames is an input error (wrong file order or `rel`): stop and say so.

**Per program second** (frames grouped by `int(t)`), report: `sec`, voice max `M`, music max `M`, SFX max `M`, music max `S`, the minimum voice-minus-music `M` over that second's speech frames (`-` if none), and flags:

| Flag | Fires when, in that second |
|---|---|
| `MUSIC-UNDER-SPEECH` | voice `M` minus music `M` < 15 dB on a speech frame |
| `BED-PEAK` | music max `S` > `aS - 3` |
| `SFX-PEAK` | SFX max `M` > `aM - 4` |
| `SFX-UNDER-SPEECH` | voice `M` minus SFX `M` < 6 dB on a speech frame |

**Footer:** the header line (`VI`, gate, `aM`, `aS`); the music bed's 10 s level span (power-average music `M` in 10 s windows, ignoring frames at or under -70, then low, high and span in dB, or "no music"); and `violations: <n> seconds` (seconds with any flag).

A one-line spot check of a single second without the full table (here second 42, music stem): `awk -F'[: =]+' '/pts_time/{t=$6} /r128.M=/{if (int(t)==42 && $NF>m) m=$NF} END{print m}' m=-999 music.r128.txt`

Violations must be 0 before export. Speech edges count: fix a flagged second by moving the bed or the hit, not by changing `rel`. The table measures level only; listen to the whole program as well.

## Sudden-drop scan (music and SFX stems)
Checks "No sudden drops" in `SKILL.md`. Run it on each music and SFX stem (bounced per track, same start and length as the program), one at a time. First write one RMS value per 50 ms (2400 samples at 48 kHz; use `n=2205` at 44.1 kHz):

```sh
ffmpeg -hide_banner -nostats -i <stem>.wav -map 0:a:0 -af "asetnsamples=n=2400,astats=metadata=1:reset=1:measure_perchannel=none:measure_overall=RMS_level,ametadata=mode=print:key=lavfi.astats.Overall.RMS_level:file=<stem>.rms.txt" -f null -
```

Then scan the log. No drop scanner ships with this plugin: if the project's `tools/` folder has one, use it; otherwise build one to this spec and save it there, never in the plugin.
- **Input:** the RMS log (a `pts_time:<t>` line, then a `lavfi.astats.Overall.RMS_level=<dB>` line, per 50 ms block). Read `-inf` as -120 dB.
- **Drop:** at block `i`, `hi` = the loudest of the blocks in the `win` ms before it (default 200 ms, so 4 blocks). Flag block `i` when `hi` is above `floor` (default -60 dBFS: a drop that starts below it is already inaudible) and `hi` minus the block's level is more than `db` (default 6 dB). After a flag, skip further flags for 0.5 s so one drop is reported once.
- **Output:** one line per drop, `<t, 2 decimals><TAB><hi> -> <level> dB`, then `drops: <n>`. Save it as `<stem>.drops.tsv`.
- **Acceptance check** (`studio` → `references/local-jobs.md` → "Prove a check before trusting it"): on a synthetic stem, a hard cut to silence must be flagged at its time, and the tail of a linear 2 s fade must be flagged too (a designed fade, classified as such). [verified, FFmpeg 9.0.2 and the macOS system `awk`, with a short scanner built to this spec]

Each line is a program time and the level before and after. Classify every one: **designed fade** (a planned, ramped fade or exit at that time), **in-source** (run the same scan on the cue's source file; the drop is there at the matching source time) or **edit** (anything else: a cut, a cue ending in a hole, a keyframe step). Edits must be 0. `floor` skips drops that start below -60 dBFS (already inaudible).
