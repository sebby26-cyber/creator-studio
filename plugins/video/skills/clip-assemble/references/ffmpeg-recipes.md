# FFmpeg recipes for clip assembly

Tested with FFmpeg 6.1 (`still_loop` and the file checks also on FFmpeg 9.0.2). Run heavy commands one at a time under the watchdog in `studio` → `references/local-jobs.md`; any command that reads only the audio of a video file needs `-map 0:a:0` or `-vn`. Every command writes a new file. Replace `<...>` placeholders, change `1920:1080` to the project's aspect (for example `1080:1920` for vertical), and set `<fps>` to the project fps (never a fixed 30; a source at another rate is converted once or logged: `episode-conform` → "Pre-flight"). All commands encode H.264 video and AAC audio with `+faststart`. `-shortest` ends the file at the shorter of picture and audio, so **check that the visuals cover the whole audio first**; otherwise the audio is cut.

## One still under audio (`still_loop`)
Reference name: `still_loop`. For audio-only platforms that need a video file.
First read the audio duration `D`: `ffprobe -v error -show_entries format=duration -of default=nw=1:nk=1 master.wav`. Then, with `D` written in as a number:
```sh
ffmpeg -hide_banner -nostats -n -loop 1 -framerate 2 -i still.png -i master.wav -vf "scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2,format=yuv420p" -c:v libx264 -tune stillimage -c:a aac -b:a 192k -t <D> -shortest -movflags +faststart out.mp4
```
- **Set `-t` to the audio duration.** [observed, FFmpeg 9] `-shortest` alone with a looped still overran the audio by 31 s. `-t <audio duration from ffprobe>` ends the file at the audio; keep `-shortest` as a second guard.
- **`-n`, not `-y`:** never overwrite an existing output; pick a new name instead.
- **Mono audio is fine:** a mono master encodes to mono AAC; no upmix is needed.
- Check the result: the output duration (`ffprobe`, "Check the final encoded file" below) should match `D` to within a frame.

## Stills slideshow
Use only after shot order and durations are resolved from the measured audio edit. Each still gets `-loop 1 -t <seconds>`; the audio is the last input.
```sh
ffmpeg -hide_banner -nostats -n -loop 1 -t 4 -framerate <fps> -i s1.png -loop 1 -t 3.5 -framerate <fps> -i s2.png -i master.wav -filter_complex "[0:v]scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2,setsar=1,format=yuv420p[v0];[1:v]scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2,setsar=1,format=yuv420p[v1];[v0][v1]concat=n=2:v=1:a=0[vc]" -map "[vc]" -map 2:a -c:v libx264 -c:a aac -b:a 192k -shortest -movflags +faststart out.mp4
```
Add one `-loop` input and one `[i:v]...[vi]` chain per still, list every `[vi]` in `concat`, raise `n`, and change `-map 2:a` to the audio input's index.

## Generated clips over the master audio
Clips are normalized to one size, frame rate and pixel format, then concatenated; the picture carries no audio, the master supplies it. To use only part of a clip, add `trim=duration=<s>,setpts=PTS-STARTPTS` at the start of its chain.
Each clip gets the same normalizing chain, written out in full per input: `scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2,setsar=1,fps=<fps>,format=yuv420p`.
```sh
ffmpeg -hide_banner -nostats -n -i c1.mp4 -i c2.mp4 -i master.wav -filter_complex "[0:v]scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2,setsar=1,fps=<fps>,format=yuv420p[v0];[1:v]scale=1920:1080:force_original_aspect_ratio=decrease,pad=1920:1080:(ow-iw)/2:(oh-ih)/2,setsar=1,fps=<fps>,format=yuv420p[v1];[v0][v1]concat=n=2:v=1:a=0[vc]" -map "[vc]" -map 2:a -c:v libx264 -c:a aac -b:a 192k -shortest -movflags +faststart out.mp4
```
If clips are shorter than the audio, hold the last frame (`tpad=stop_mode=clone:stop_duration=<s>`) or ask for more footage; do not let `-shortest` silently truncate the narration.

## Burn in captions
Needs an FFmpeg build with the `subtitles` (libass) filter. Check first:
```sh
ffmpeg -hide_banner -filters | grep -w subtitles
```
If it is missing, follow the `studio` skill's `references/dependencies.md` (on macOS, `ffmpeg-full`). Then:
```sh
ffmpeg -hide_banner -nostats -y -i in.mp4 -vf "subtitles=captions.srt:force_style='MarginV=60'" -c:v libx264 -c:a copy out_captioned.mp4
```
Keep the `.srt` path simple (no colons or quotes); copy it next to the video if needed. Captions use the actual audio-edit timings.

## Check the final encoded file
This section is the one home of the encoded-file checks for any assembler (`clip-assemble`, `filmcraft` or another editor's export).
```sh
ffprobe -v error -show_entries stream=codec_name,width,height,r_frame_rate:format=duration -of default=nw=1 out.mp4
ffmpeg -hide_banner -nostats -threads 4 -i out.mp4 -vn -af ebur128=peak=true -f null - 2>&1 | sed -n '/Summary:/,$p'
```
Compare duration with the audio-edit record, then check loudness and true peak against the project target (encoding can change peaks).

**Audio packet gaps** (should report 0).
- **Measure:** the audio packets' timestamps, in order. The nominal packet duration `n` is the most common packet `duration_time` (about 21.3 ms for 48 kHz AAC).
- **Gap:** any step between consecutive packet timestamps that exceeds `n` by more than 5 ms. Report each as `gap at <previous pts + n>: <step − n> s`, then the count.
- **Pass:** 0 gaps. Save the output under the version's `evidence/` and the count in `status.md`. Before trusting the scan on a new build or format, prove it on a planted gap and a clean encode (as in the test below): `studio` → `references/local-jobs.md` → "Prove a check before trusting it".
- Do not compare a packet's start with the previous packet's start plus its `duration_time`: MP4 and MKV store the jump as a longer duration on the packet before the gap, so that test always reads 0.

Nominal duration, then the scan (one line each):
```sh
ffprobe -v error -select_streams a:0 -show_entries packet=duration_time -of csv=p=0 out.mp4 | sort | uniq -c | sort -rn | head -1
ffprobe -v error -select_streams a:0 -show_entries packet=pts_time -of csv=p=0 out.mp4 | awk -v n=<nominal> 'NR>1 && $1-p-n > 0.005 {printf "gap at %.3f: %.3f s\n", p+n, $1-p-n; g++} {p=$1} END {print "audio packet gaps: " g+0}'
```
Tested with FFmpeg 9.0.2 and the macOS system `awk`: a synthetic 4 s H.264/AAC MP4 with a 0.3 s audio timestamp jump at 2.0 s (`asetpts='if(gte(T,2),PTS+0.3/TB,PTS)'`) printed `gap at 2.005: 0.300 s`, the same file remuxed to MKV too (nominal 0.021 s there), and a clean encode printed 0. The decoder does not turn such a gap into silence (the dropout check below did not see it); it shortens the audio and shows up as drift after the gap.

**Dropouts** (stretches of near-silence in the exported audio):
```sh
ffmpeg -hide_banner -nostats -threads 4 -i out.mp4 -map 0:a:0 -af silencedetect=noise=-60dB:d=0.05 -f null - 2>&1 | grep -E 'silence_(start|end)'
```
Every dropout must match a designed silence in the plan (a pre-twist silence, the final ~0.5 s); any other is a defect. Tested on a synthetic export: a 0.2 s hole at 3.0 s was reported as `silence_start: 3.02`, `silence_duration: 0.18`.

**Playback stutter versus a file fault.** An editor's live playback of many tracks can skip or blank audio that is fine in the file. Before chasing a reported glitch, play the exported file in an ordinary player at that timestamp and run the two checks above. A glitch that is not in the export and not in the measurements is a playback artefact: note it, do not re-edit for it.

**End padding versus drift.** Read both stream lengths:
```sh
ffprobe -v error -show_entries stream=codec_type,start_time,duration -of csv=p=0 out.mp4
```
A 10-20 ms difference between audio and video at the end is encoder padding (AAC priming and frame rounding), not drift, when sync is exact at the end: check a late sync point (the last word against its caption or shot cut, a sound hit against its picture event, the sonic logo against its anchor) on frame grabs. Do not "fix" padding. Drift is an offset that grows along the program; measure it at an early and a late sync point and fix it at its source (frame rate, sample rate or a retimed clip).
