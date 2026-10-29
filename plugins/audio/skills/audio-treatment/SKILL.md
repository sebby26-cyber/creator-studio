---
name: audio-treatment
description: "Apply a named tone/level treatment (EQ, gain, etc.) defined in the project file at audio.treatments.<name>, or given explicitly by the user, with FFmpeg into a separate file, then measure it (ebur128) and report. Plug-and-play on the whole program or on fragments. Also runs the automatic per-chunk opener level dip before the join, documents an example de-harsh treatment, holds the voice lock rule (after polish the voice is never processed again) and the voice floor rule (never gate the floor between words for an episode with picture). Zero credits."
---

# Audio treatment

Applies a named treatment exactly as written: from the project's `audio.treatments.<name>`, or settings the user gives explicitly. Never invent settings, and never carry one project's treatment to another.

- **Where:** whole stitched program by default (after join, mix and master) for an audio-only program. For an episode with picture, a voice treatment (for example "De-harsh treatment" below) runs on the joined, paced voice before the voice is locked, never on a mix ("Voice lock" below). Per fragment or clip, before the stitch, only when the treatment or the user says so. If the project gives source fragments, join them in their intended order first.
- **Separate output:** write a new file in the treatment's format (e.g. 24-bit WAV). Never overwrite the source.
- **Only the listed filters:** no added compression, reverb, voice regeneration or timing change unless the treatment says so.
- **Closing fade:** follow the treatment (e.g. keep an existing fade; add one only if absent).
- **No re-master:** do not re-run `audio-master` or normalize afterwards unless the treatment says so. The level change is intentional.
- **Measure and report:** integrated LUFS, loudness range (LRA) and true peak of the finished file.

```
ffmpeg -i <in> -map 0:a:0 -af "<treatment filter_chain>" -c:a <codec> <out>
ffmpeg -i <out> -af ebur128=peak=true -f null -
```
Read the `Summary:` block of the second command (`I`, `LRA`, `Peak`). `-map 0:a:0` also takes the audio stream from a video.

## In Creator Studio
- **Takes:** source audio (or a video's audio stream) and a named treatment.
- **Makes:** the treated file plus its loudness report (integrated LUFS, LRA, true peak).
- **Tool:** FFmpeg; Rubber Band only for a rate taper, and a speech-to-text tool (whisper.cpp) for the end check (checks and installs: `studio` → `references/dependencies.md`). Zero credits.
- **Save:** put the treated file next to the source in the production folder and add a line to `status.md` (treatment name, filter chain, measurements). If it is a final, write its sidecar (see the `studio` skill → `references/output-sidecar.md`). The user's listen gate follows.

## De-harsh treatment (narrated documentary, example)
[observed] A treatment that fixed a harsh narration read in one long-form narrated documentary, offered as a named example the user can adopt into `project.json` (`audio.treatments.deharsh`); as with every treatment here, nothing runs unless the project or the user names it. It pairs with the punctuation pauses in `pace-and-pause` (the rushed half of the same problem).
- **De-esser above 4.5 kHz:** ratio 3, at most 5 dB of reduction, acting only on sibilant peaks.
- **High shelf:** -2 dB at 7 kHz.
```
ffmpeg -hide_banner -nostats -n -i voice_paced.wav -map 0:a:0 -af "adynamicequalizer=threshold=<t>:dfrequency=4500:dqfactor=0.7:dftype=highpass:tfrequency=4500:tqfactor=0.7:tftype=highshelf:mode=cutabove:ratio=3:range=2.5:auto=off,highshelf=f=7000:g=-2" -c:a pcm_s24le voice_deharsh.wav
```
- **`range` is not the cut in dB.** With `tftype=highshelf` the maximum cut is about **twice** `range` in dB: `range=2.5` caps the de-esser at about 5 dB (the cap above), `range=5` at about 10 dB. [verified, FFmpeg 9.0.2, synthetic white noise above 6 kHz at -12.5 dBFS RMS, threshold far below the signal] `range` 1 / 2.5 / 5 / 10 cut the band by 1.9 / 4.9 / 9.7 / 19.1 dB, the same as a plain `highshelf` of twice that gain (`g=-5` matched `range=2.5`). With `tftype=bell` the same settings cut far less on that band, so the factor of two holds for the high shelf only. Once the signal is well over the threshold, `ratio` (1.5 to 30) did not change the cut: `range` is what limits it. Re-measure on your build before trusting the cap.
- **`threshold` is a detector level, not dBFS** (0-100, compared with the smoothed output of the detection filter). On the same test noise, 0.01 to 0.1 cut fully and 0.2 did nothing, although the noise peaked near -8 dBFS. Start near 0.02 and raise it until only sibilant peaks trigger, listening each time. `auto=on` barely acted in testing; use a fixed threshold.
- Measure and report as for any treatment; compare a sibilant-heavy passage before and after by ear.

## Voice lock
This is the one home of the voice lock; other skills point here. Voice polish runs in this order on the joined narration: pause inserts and rests (`pace-and-pause`), then the tone treatment (for example "De-harsh" above), then a level check. After that the voice is **locked**: one file, hashed and recorded in `status.md` as the locked voice. It is never processed again: no further treatment, gain, stretch, fade, normalization or re-render in mixing or assembly. In the editor it is only placed, moved and split at silences; the one edit allowed at a split is the splice fade pair (`pace-and-pause` → "Splice fades at every insert or split"), which shapes the edge of a silence and is not processing (`filmcraft` → "Assembly split"). Polishing changes the voice's length, so everything timed against the old voice (timing table, shots, cues, captions, markers) is retimed to the locked voice, never the other way round. A later fix to the voice is a new version that goes through the whole polish order again and re-locks, and its retime is recorded.
- **The timing table comes from the lock.** It is measured from the locked file (sentences, words, silences, and the insert table `pace-and-pause` hands on), and it names the lock's hash. Shots, cues, markers and captions are built from it, never from an earlier take.
- **Any change invalidates the lock.** An inserted or moved pause, a re-rendered or re-spliced line, a new treatment: any change to the voice after the timing table is made creates a **new lock version** (`voice-lock-v<N>`, new hash, a `status.md` line saying what changed). That version re-derives its timing table from the new file, and every downstream table (`timeline.json`, cues, level keys, markers, captions) is re-mapped through `episode-conform`'s clock map (inserts, then the breather ripple) in one pass. Never retime rows one at a time by hand, and never with a one-off script outside the project's `tools/`. Until the re-map runs, every table built from the old lock is stale (`episode-conform` → "Pre-flight").

## Voice floor
Keep the provider's own floor between words. For an episode with picture, never gate, expand or trim it down to digital silence in any treatment: the floor is what keeps a pause sounding continuous against the bed, and it is the source of the room tone that fills inserted pauses (`pace-and-pause` → "Fill every insert inside a sentence with room tone"). [observed, one long-form narrated documentary, raw ElevenLabs takes] a generated voice can already fall very low between words with no room tone of its own, so each pause reads as a hole against the bed; the fill rule covers the inserts, and nothing should push the floor lower.

## Split-map assembly (join per split map)
[observed] An assembly pattern from one long-form sleep production, generalized; details beyond that pattern are marked. Assembly consumes the split map the direction job emits (`elevenlabs-voice-director` -> Step 6 "Split map"; hard rule and tradeoff: `studio` -> `references/elevenlabs-tagging.md` -> "Pause rests, splits and the split map"). This is the "join" step of the chain order below.
- **Joins land on the map's boundaries**, in boundary order. Never cut or join elsewhere.
- **Owed silence at each join**: measure the delivered gap at the boundary (chunk tail plus next chunk head, or the tag's gap), then add `max(0, target_s - measured)` (the `audio-post-mix` pause rule), so the map's `owed_s` is an estimate that the measurement replaces. An entry with `target_s: null` (split granularity only, `elevenlabs-voice-director` → Step 6 "Split granularity") gets no insert here; `pace-and-pause` sets that rest.
- **Rest fill, crossfaded**: fill per the project's rest-fill setting (e.g. `voice.sleep.rest_fill.source`: `room_tone`, `from_program` or `none` for digital silence). Joins between sentences or paragraphs may take digital silence as the project's opt-in; a pause inside a sentence is always room tone (`pace-and-pause` → "Fill every insert inside a sentence with room tone"). That includes every in-sentence join (a `phrase` entry, with `audio.chunking.split_at: "phrase"`): filled per `room_tone_source` from the take floor at that join (`from_program`, or the project's room-tone file), never digital silence, even where the project's rest fill is `none`. When tone is used, crossfade into and out of it. Never a hard splice mid-take. Crossfade length is the project's; none is recorded here.
- **Relative levels preserved**: no per-chunk normalization. Loudness stays post-stitch, on the whole program (below). A measured level step that needs fixing is a separate, logged level-match (voice-director 4a-5), not normalization [inferred].
- **Not the softness pass's fragment mode**: its fragment join is direct concatenation with no inserted silence or crossfade. Assemble per the map first, then pass the joined program.
- **QC: silence-detect audit**: ffmpeg `silencedetect` (same settings as the ladder: -40 dB, >=0.25 s) at each boundary; report delivered gap vs `target_s` per entry. No tolerance is measured or project-approved; [inferred] starting point: flag any deviation above ~0.3 s (the ladder's single-take noise floor) for listening, and treat a breath as speech, not silence.
- **Audit before the rate ramp** [inferred]: the ramp stretches the gaps too, so audit the joined program against the map's pre-stretch targets. After the ramp, a target only holds scaled by the local stretch factor from the `--timemap`; the standing intro (1.00x) is unscaled.

## Opener level dip (per chunk, before gain-match and join)
Automatic, free, per raw chunk. It corrects a chunk whose opening runs louder than the rest of the chunk (on tagged sleep reads, the opener's quiet tag fades and the following sentence rebounds). [inferred, simulated on one long-form episode; confirm by ear] It is the logged level-match step for that case, not normalization.
1. **Measure each chunk.** Opener = integrated LUFS of the first ~10 s; body = integrated LUFS from ~10 s to the end (`ebur128`, same settings for both). Lift = opener − body.
   ```
   ffmpeg -i chunk.wav -t 10 -af ebur128 -f null -           # opener I
   ffmpeg -ss 10 -i chunk.wav -af ebur128 -f null -          # body I
   ```
2. **Trigger.** Only when lift ≥ +1.0 LU. Otherwise leave the chunk untouched and log "no dip".
3. **Gain.** Lower the opener by `(lift + 0.5)` dB. The dip region runs from the chunk start to the first measured pause (silencedetect, -40 dB, ≥ 0.25 s) at or after ~10 s, so it ends at a sentence gap, never mid-word.
4. **Ramps inside pauses, never across speech.** The return ramp to 0 dB (the project's crossfade length, else 50 ms) sits wholly inside that pause, centred on its midpoint. The dip starts at the chunk's first sample (head quiet), so no ramp crosses speech there either. If no pause of that length falls between ~10 s and ~20 s, do not dip; flag the chunk for listening.
   ```
   ffmpeg -i chunk.wav -af "volume=eval=frame:volume='if(lt(t,R0),D,if(lt(t,R1),D+(1-D)*(t-R0)/(R1-R0),1))'" -c:a pcm_s24le chunk_dip.wav
   ```
   `D` = 10^(−(lift + 0.5)/20); `R0`/`R1` = ramp start/end inside the pause.
5. **Re-measure and log.** Re-run step 1 on the output; log per chunk: lift before, gain applied, region end, ramp position, lift after. Write a separate file; never overwrite the raw chunk.
6. **Order.** Dip first, then the pre-join gain-match (`audio-post-mix` → "Gain-matching chunks"), then the join. The final (outro) chunk is still dipped if it qualifies; only the gain-match exempts it.

## Rate taper and chain order (sleep narration practice)
[observed] Practice from one long-form sleep production; a timing change, so it runs only when the project's treatment or the user asks for it. No helper script ships for the ramp.
- **Rate taper:** rubberband, R3 engine, driven by a `--timemap` file describing a smooth ramp (the read slows gradually; no steps). ElevenLabs v4 has no `speed`, and a per-call speed would be constant anyway, so a taper is always post.
- **Anchor:** the ramp starts at the story start. The standing intro is excluded and stays at 1.00x.
- **Pauses stretch with speech:** the stretch scales the silences too; re-check pause timing after it (split-map targets are pre-stretch, above).
- **Chain order:** opener level dip per chunk (above) -> gain-match -> join (per split map, above; QC audit here) -> rate ramp -> soften x2 ("Softness pass" below, no added fade) -> sleep taper ("Sleep taper" below).
- **Re-measure E after the stretch:** the sleep taper anchors on E (end of the last spoken word), which moves when the program is stretched. Pass the post-stretch E to the taper.
- **Loudness stays post-stitch and separable:** every level step runs on the whole stitched program and can be undone or re-run independently. A reference episode: -19.2 LUFS (program) -> -26.3 (after soften x2) -> -26.7 (after the sleep taper).

## Softness pass (spec)
The final-stitch softness pass (project key `audio.treatments.softer`; validated by ear on one long-form sleep production, 2026-10; whether it is a project's standing final step is the project's decision). No script ships with this plugin: if the project's `tools/` folder has a softness tool, use it; otherwise build one to this spec and save it there, never in the plugin.

One pass is exactly `equalizer=f=3000:t=q:w=0.8:g=-2,highshelf=f=6500:g=-1.5,volume=-3.3dB` (bell cut 3 kHz -2 dB Q 0.8, high shelf 6.5 kHz -1.5 dB, gain -3.3 dB) on the whole stitched program and nothing else: no compression, reverb, regeneration, timing change or re-normalization. **Recommended standard: two passes** (the chain written twice, `chain,chain`, in one FFmpeg run), delivered as a 24-bit WAV master (`<name>_soft2.wav`); the project file (`passes`, `filter_chain`) decides.

```
ffmpeg -i assembled.wav -map 0:a:0 -af "<chain>,<chain>" -c:a pcm_s24le assembled_soft2.wav
```

A project tool built to this spec must:
- **Input:** one stitched program, or fragments joined first in the given order (or from a list file, one path per line) by direct concatenation at 48 kHz mono by default, no inserted silence. That fragment join is not a split-map join (above). Optionally keep the untreated join. Join rate and channels apply to fragment joins only; a single input keeps its own.
- **Passes:** a whole number, at least 1. Count the passes the input already carries: the total must match the treatment. If the program is already softened, undo the prior passes first (the exact inverse chain) or apply fewer.
- **Closing fade:** measure the tail (RMS of the three 1 s windows over the last 3 s, plus the final 0.2 s). A final 0.2 s at or under -50 dBFS, or levels falling steadily by at least 8 dB across the last 3 s into a final 0.2 s at or under -35 dBFS, counts as an existing fade and is preserved. Otherwise add one gentle `afade` out (default 3 s) after all passes, unless told not to. **Recommended: never add it** where the E-anchored sleep taper (below) is the closing fade.
- **Never overwrite:** sources are read only; refuse if the output exists, if it would overwrite an input, or if its folder does not exist. Exit non-zero on any refusal or a missing `ffmpeg`/`ffprobe`.
- **Report:** the fade verdict, then integrated LUFS, LRA and max true peak of the output (`ebur128=peak=true`). Do not re-normalize. A reference episode for scale: -19.2 LUFS program, -22.8 after one pass, -26.3 after two.
- **Acceptance check:** two passes on a reference episode's unsoftened program nulled against its approved double-soft master at -128.9 dBFS peak / -143.2 dBFS RMS (24-bit rounding level, same length). A rebuilt tool should reproduce an approved master the same way before it is trusted.

Zero credits. Then save and log as above; the user's listen gate follows.

## Sleep taper (spec)
The closing fade-out / sleep taper (project key `audio.treatments.closing_taper`; the defaults are a "Standard, -4 dB floor" spec approved by ear on one long-form sleep production, 2026-10). Run it **after** the double softness pass, on `<name>_soft2.wav`; its output is the final master. Same tooling rule as above: a project `tools/` tool, built to this spec, never a plugin script. Everything is anchored to **E, the end of the last spoken word**, not to the end of the file. Defaults (a project's own `closing_taper` values replace them):
- **Taper:** `hsin` S-curve over 360 s from E - 20 - 360 s down to a -4 dB floor reached at E - 20 s, held flat through the closing lines (`afade=t=out:st=<E-380>:d=360:silence=<10^(floor/20)>:curve=hsin`).
- **Darkening:** a 4 kHz -4 dB high shelf crossfaded in dry/wet over the same 360 s (split the signal; fade the dry branch out and the shelved branch in with `hsin`; `amix=normalize=0`). Optional.
- **Room tone:** a 1.5 s `qsin` fade from E + 0.5 s.
- **Ghost-word guard:** mute from the first digital-silence run (at least 0.2 s, `silencedetect=n=-140dB`) within 3 s after E, a hard mute inside digital zero so no click; if there is none, from E + 1.2 s with a 20 ms fade. A given time or "off" may replace it. It exists because ElevenLabs can leave a faint ghost word after the last line (a reference episode: a ghost "sleep" about 1.3 s after "Sleep.").
- **Pad:** trailing digital silence = the project's `audio.pace.end_tail_s` (warn when a fallback is used).
- No compression, no normalizing. A separate 24-bit WAV; never overwrite (same refusals as above).
- **E:** pass it from the transcript. If the tool detects it (silencedetect -50 dB / 0.4 s over the last 30 s; segments more than 20 dB below the loudest are ghost candidates, not speech; rounded to 10 ms), it must print a warning and the ghost candidates, and E must be confirmed before use. Refuse when E does not fit (0 < E < duration and E - lead - taper >= 0).
- **Report:** integrated LUFS, LRA, true peak; body RMS (default window 300-700 s) against final-word RMS with the gap in dB (rule of thumb: no more than ~20 dB, confirmed by ear at bedside volume); and the ghost check (source level after E, and that the output is digital zero from the mute point, else FAIL).
- **Then verify by ear and transcript** that nothing but silence follows the final word (for example `whisper-cli` on the last 10 s, or `mlx_whisper` on Apple Silicon).
- **Acceptance check:** on a reference episode's double-soft program, both the transcript-confirmed E and auto-detection reproduced that episode's approved tapered master byte for byte (zero residual): -26.7 LUFS, LRA 5.9 LU, -8.0 dBTP, final-word gap 20.2 dB.
- Needs `ffmpeg`/`ffprobe`. Zero credits.
