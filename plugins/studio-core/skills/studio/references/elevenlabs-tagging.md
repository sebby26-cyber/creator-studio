# ElevenLabs tagging and route controls

Evidence snapshot: 2026-10-17. [verified] means official documentation facts gathered by the maintainers (OpenRouter sources fetched that date); [observed] means a reported production run or maintainer experiment; [maintainer-observed] means a maintainer’s first-hand listening observation, not official guidance; [inferred] means repository/practitioner guidance; [unverified] means not established by those sources. [measured] / [measured-controlled] mean instrument measurements of renders (voice basis and dates stated per section), not listening judgements; they come from one or two voices, mostly single takes: verify on your voice and model. This documentation edit used no network or paid calls. Updated 2026-10-19 with a second-voice v3/v4 probe set and the split-map handoff contract; updated 2026-10-20 with long-form v4 pause findings (designed rests become splits). A project's own evidence, IDs and decisions live in that project's docs, not here.

## Select the profile, then direct

Repository rule: the model ALWAYS comes from the project's `voices[].model`, never a code/provider default. Resolve an explicit production choice into the voice profile for its scope and record it in `status.md`; a production override need not rewrite the saved project. `models.voice` supplies project selection constraints/setup choices, not an implicit replacement for a cast voice's model. If no profile model is resolved, obtain a choice before rendering.

[verified] ElevenLabs recommends v4 for new work. v3 remains valid for a project/voice that selects it. Both accept the same free-form natural-language tags in square brackets. Claims that the same tags work on both models or scripts carry over unchanged describe syntax support only. v4 follows tags more reliably, including delivery outside a voice's training, but results may be suboptimal: a serious voice may respond poorly to playful direction. PVCs are not fully optimized for v3. Sources: [help center](https://elevenlabs.io/docs/help-center/product/core-capabilities/text-to-speech/how-do-audio-tags-work-with-eleven-v3-and-v4), [models](https://elevenlabs.io/docs/overview/models).

Use [models/elevenlabs.json](models/elevenlabs.json) `tags_supported` before sending tagged text on either route (map the selected OpenRouter id explicitly to its upstream model; do not change the selected id). [unverified] Exact OpenRouter route ids beyond `elevenlabs/eleven-v4`, availability, voices and prices require live discovery; the supplied announcement establishes model-family limits. `models/openrouter.json` records these families by upstream id without inventing route slugs. Model availability in an MCP schema or web UI must be checked separately; direct API parameters are not automatically connector parameters.

## Syntax and performance

[verified] Use square brackets, never parentheses or braces, immediately before the affected words. Emotion carries across the line; add a tag when delivery changes. Combine qualities with commas in one bracket: `[whispering, fearful] Stay close.` Sources: [best practices](https://elevenlabs.io/docs/overview/capabilities/text-to-speech/best-practices), [audio tag list](https://elevenlabs.io/blog/elevenlabs-audio-tags-list), [help center](https://elevenlabs.io/docs/help-center/product/core-capabilities/text-to-speech/how-do-audio-tags-work-with-eleven-v3-and-v4).

[measured] Scope caveat (one v4 voice, 2026-10): a DELIVERY tag such as `[quietly]` lasted about one sentence and reset across a `[long pause]` x2, so "carries across the line" must not be read as "carries across a paragraph or chunk". See **Delivery-tag scope and reset** below.

| Family | Examples | Model guidance |
|---|---|---|
| Delivery | `[whispers]`, `[shouts]`, `[softly]` | [verified] v3/v4 share syntax; match the voice, test unfamiliar deliveries. |
| Emotion | `[curious]`, `[excited]`, `[crying]` | [verified] Carry the emotion across the line until delivery shifts. |
| Non-verbal | `[laughs]`, `[sighs]`, `[gasps]`, `[clears throat]` | [verified] v4 may also add unprompted stammers/groans. Listen for unwanted additions. |
| Pacing | `[pause]`, `[long pause]`, `[slowly]`, `[rushed]` | [verified] Less deterministic than SSML; use post for exact silence. |
| Sound effects | `[applause]`, `[gunshot]` | [verified] Less consistent and may be misread. Describe vocal quality explicitly (`[low, gravelly voice]`) to avoid SFX misreads. Use `sound-render` for separately timed effects. |
| Accent / character / singing | Accent phrases, `[sings]` | [verified] Experimental; test per voice. |

[verified] Ellipses add pause/weight, CAPS add emphasis, dashes signal interruption. [inferred] Prefer audible qualities over body-action descriptions, sparse tags at beat changes, and no clashing directions on the same words. “1-3 tags per sentence” is third-party practitioner guidance, not a vendor quota. [unverified] Keeping tags in English is a convention here; no official localization guidance was supplied. Sources: [best practices](https://elevenlabs.io/docs/overview/capabilities/text-to-speech/best-practices), [audio tag list](https://elevenlabs.io/blog/elevenlabs-audio-tags-list).

[verified] Multilingual v2, Flash and Turbo v2/v2.5 do not use audio tags. [inferred] Unsupported tags would likely be read aloud: strip direction tags or convert their intent into punctuation and supported settings, preserving spoken words. [verified] SSML `<break>` is unsupported on v3/v4; older models accept breaks up to 3 seconds. `<phoneme>` is supported only on `eleven_flash_v2`. [unverified] Slash-delimited IPA as a v3/v4 control is not established here; prefer a checked respelling. OpenRouter accepts `pronunciation_dictionary_locators`, but dictionary behavior on v3/v4 is [unverified]. Source: [best practices](https://elevenlabs.io/docs/overview/capabilities/text-to-speech/best-practices).

## Model changes and listening review

[maintainer-observed] Shared tag syntax does not mean shared sound: delivery is model-specific. Never assume a tuned v3 script sounds the same on v4, or vice versa.

Repository rule: when a project switches or mixes v3/v4, re-audition every tagged line before approving finals, including at minimum delivery tags (`[whispers]`, `[shouts]`, `[softly]`) and emotion tags. A model switch on an approved production requires the user's listening gate again; never record an approval on the user's behalf.

Record the actual render model per take in its existing `<file>.json` sidecar and in `status.md`, following [output-sidecar.md](output-sidecar.md) and [production-handoff.md](production-handoff.md). Compare those recorded models with the resolved profile on a switch or mixed-model handoff; flag affected tagged takes `pending_review` with re-listening as the next action. Preserve their original render model and prior review history; a new profile choice does not relabel an old take.

## Per-model tag notes

Record observed differences here with their evidence label; do not infer unreported details.

### Measured v3 vs v4 (controlled set, 2026-10-18)

[measured-controlled] These are instrument measurements of 26 short renders, not listening judgements; "consistent with" describes signal properties only. Each tag column is the change versus the untagged control on the SAME model (tag minus none), so the model's own baseline is removed. Two takes each of none, `[whisper]` and `[quietly]` give a run-to-run noise estimate; a v3/v4 difference is called above noise only if the v4 effect minus the v3 effect exceeds the largest take-to-take gap seen in those 6 pairs. Tags other than those three have ONE take per model: [indicative] only. Small sample (3 pairs per model), so the noise ceiling is itself rough.

Noise (take-to-take gap, mean / max): LUFS 0.85 / 3.0 dB; RMS 1.2 / 2.5 dB; voiced fraction 3 / 7 points; share of energy above 4 kHz 4 / 14 points; duration 0.16 / 0.48 s.

| Tag | LUFS change v3 / v4 (dB) | Voiced change v3 / v4 (points) | >4 kHz share change v3 / v4 (points) | Above noise? |
|---|---|---|---|---|
| `[whisper]` (2 takes) | -3.8 / -2.7 | -14 / -62 | +1 / +38 | Yes for voiced, >4 kHz, centroid; no for LUFS |
| `[quietly]` (2 takes) | -3.1 / -4.5 | -8 / -68 | +1 / +64 | Yes for voiced, >4 kHz, RMS; no for LUFS |
| `[softly]` | -0.6 / -3.7 | -2 / -59 | 0 / +42 | Yes, indicative (1 take) |
| `[shouts]` | +0.2 / +8.1 | -1 / +10 | 0 / -3 | Yes for loudness (v4 only), indicative |
| `[excited]` | -0.6 / +3.3 | -4 / +5 | 0 / -1 | Marginal (LUFS gap 3.8 vs 3.0 noise) |
| `[sad]` | -0.8 / -0.5 | -6 / -9 | 0 / +4 | No |
| `[laughs]` | -2.3 / -1.4 | -16 / -15 | 0 / -1 | No (both models ~0.7 s longer) |
| `[sighs]` | -0.1 / -0.9 | -9 / -20 | 0 / +3 | No for loudness; voiced marginal |

Baseline (no tag): v4 measured 8.1 dB quieter than v3 (-21.3 vs -13.2 LUFS), true peak -8.4 vs -1.0 dBFS, voiced 73% vs 87%, trailing silence 0.18 vs 0.00 s; well above the control pair's own gap (LUFS 0.3-0.5 dB). Level-match across models before comparing or mixing.

Notes (all [measured-controlled] unless marked):
- `[whisper]`, `[quietly]`, `[softly]` on v4: voiced frames collapse to roughly 10%, 5% and 14%, with 36-72% of energy above 4 kHz and a spectral centroid of 3-5 kHz (control ~0.5 kHz). Consistent with a breathy, unvoiced delivery. On v3 the same tags moved voicing by 2-14 points and left the spectrum unchanged: consistent with a quieter but still voiced read. Direction is from one voice and one sentence; do not generalize beyond it. Flag 2026-10-19: on a second voice in-passage, v3 `[quietly]` / `[whispers]` showed no measurable level change at all (quiet probe below), so the v3 "quieter" result does not hold for every voice/position.
- `[shouts]`: v4 +8 dB over its own control (consistent with a louder delivery; it reached the same absolute level as v3's control); v3 was indistinguishable from its control on loudness and spectrum (1 take each).
- `[sad]`, `[laughs]`, `[sighs]`: no v3/v4 difference above noise on loudness or spectrum. `[laughs]` and `[sighs]` lengthened both models by ~0.6-0.8 s, consistent with a non-speech sound or pause; whether the tag text itself was spoken was not checked.
- `[long pause]` ("The light is almost gone. [long pause] Now it is dark.", 1 take each): silence between the sentences measured 4.8 s on v3 and 1.8 s on v4. Not a fixed duration and not comparable across models.
- Spelling: the earlier renders used `[whisper]`; ElevenLabs' tag list spells it `[whispers]`. Both are free-form tags; re-check on your voice whichever you use.

[maintainer-observed] Earlier separate renders of longer text (voices/settings unconfirmed; maintainer measurements, 2026-10-18), which agree in direction with the controlled set:
- `[whisper]`: v4 opened more whisper-like than v3 (first sentence ~15% voiced frames vs ~29%; first sentence 8 dB below the rest vs 4 dB). Neither held the whisper past the first sentence: repeat the tag where it should persist.
- `[quietly] ... [long pause]`: v4 rendered ~11 dB quieter than v3 (-30.2 vs -19.3 LUFS), with more energy above 4 kHz (19% vs 3%) and fewer voiced frames (25% vs 66%). v3 faded toward the end; v4 stayed level. Trailing silence after `[long pause]` ~1.46 s on v3 vs ~0.94 s on v4 (not fixed).

### Measured pause ladder, second voice, in-passage (2026-10-18)

[measured] Instrument measurements, not listening judgements; ONE take per condition, so differences under ~0.3 s are within noise. **A short probe: in a long-form v4 render the same tags overshot these values and drifted (see "Pause rests, splits and the split map"), so this ladder is not a v4 timing tool.** Route: OpenRouter `POST /api/v1/audio/speech` via `speech-render` (curl), one library voice passed as a raw voice id (accepted; not proven to be the same voice as the same-named direct-account voice), `mp3`, no `instructions`/`speed`. v3 sent `provider.options.elevenlabs.voice_settings.stability=0.5` (accepted; mapping to the "Natural" preset unproven); v4 used route defaults. Passage: five short calm sentences with four tag slots A-D between them, 555 characters in 4 renders. Delivered silence = ffmpeg `silencedetect` (-40 dB, >=0.25 s); Whisper word-gaps blur into silence, so ffmpeg is primary.

| Slot | v3 tag | v3 silence (s) | v4 tag | v4 silence (s) |
|---|---|---|---|---|
| A | `[long pause]` x1 | 2.34 | `[short pause]` | 1.08 |
| B | `[long pause]` x2 | 2.96 | `[pause]` | 1.04 |
| C | `[long pause]` x3 | 2.50 | `[long pause]` x1 | 1.11 |
| D | none (native control) | 1.07 | `[long pause]` x2 | 1.60 |
| in-sentence comma | | 0.42 / 0.51 | | 0.32 (second comma <0.25) |

- v3: the native sentence gap is ~1.07 s; each `[long pause]` run added ~1.3-1.9 s on top. Stacking x1 -> x2 -> x3 did NOT scale (2.3, 3.0, 2.5 s): treat >2 stacked tags as no gain.
- v4: `[short pause]`, `[pause]` and `[long pause]` x1 are indistinguishable in this probe (~1.0-1.1 s, one take each); only `[long pause]` x2 stepped up (1.6 s). v4 has no no-tag control slot in this set. Never double pause tags on v4 (long-form overshoot, below); on v3 never stack more than 2.
- v4 variance [measured, one v4 voice, 2026-10-19]: across identical takes the delivered `[long pause]` x2 gaps ranged 1.3-2.4 s (~0.5 s take-to-take spread); in a long-form episode the median was ~2.5 s and it lengthened over the episode. The ladder figures are typical values from single takes, not fixed durations.
- v3 -> v4 mapping: a v3 tag-timed rest has no reliable v4 tag equivalent: on v4 every designed rest longer than a sentence break becomes a split-map entry (**Pause rests, splits and the split map**). In this probe v4 `[short pause]` = `[pause]` = `[long pause]` x1 (~1.1 s).
- This disagrees with the single-sentence figure above (`[long pause]` 4.8 s v3 / 1.8 s v4, voice `sarah`, one sentence): pause length depends on voice and position. Neither is a constant; re-measure per voice.
- Transcript (Whisper): no tag was spoken aloud in any of the four renders; the plain sentences came back (v4 dropped the Oxford comma only).

Quiet probe: two short sentences, the first tagged `[quietly]`, the second `[whispers]` (one take per model):

| Model | `[quietly]` | `[whispers]` | Gap | Level (dBFS, proxy) |
|---|---|---|---|---|
| v3 | consumed, not spoken; no level/noise change vs untagged v3 (-20.4 vs -20.2) | consumed, not spoken; no change beyond v3's natural end-of-passage drop (-23.4 vs -23.8) | 1.33 s | -20.4 / -23.4 |
| v4 | consumed, not spoken; ~12 dB quieter than v4's untagged main take (-35.3 vs -23.1) and noisier (breathy) | same level as the `[quietly]` line (-35.6): no step between the two tags in one take | 0.60 s | -35.3 / -35.6 |

Reading: v3 behaviour is consistent with both tags being ignored (cannot be proven from level alone); v4 `[quietly]` is already breathy/whisper-like and `[whispers]` adds nothing measurable on top. Whether either "sounds like a whisper" is a listening question. Matches the user-observed note that v4 `[quietly]` is a true whisper.

[measured, second voice] v4 `[quietly]` across the 2026-10-18/19 probes: ~9-12 dB below the untagged control; voiced fraction 0.05-0.14 vs ~0.5-0.7 untagged; >4 kHz energy share 34-43% vs ~10%. On its own sentence it acts as a whisper; `[whispers]` adds nothing beyond `[quietly]`. The same voice untagged also ran ~8 dB quieter on v4 than on v3, matching the controlled set. Use: [measured, one v4 voice, one long-form episode, 2026-10] as a paragraph-start hold tag re-applied through a sleep read it measured calmer and still usable (`elevenlabs-voice-director-sleep`); whether that reads as a whisper chain over a long listen is a listening question. `[whisper]` / `[softly]` stay out of a sleep body.

### Delivery-tag scope and reset, one v4 voice (2026-10-19)

[measured] One voice, `eleven-v4` over OpenRouter, cases A (two takes), B and C; per-sentence level vs an untagged control on the same model; instrument measurements, not listening judgements.

| Case | Tag position | Tagged sentence | Following sentence(s) |
|---|---|---|---|
| A (2 takes) | `[quietly]` text-initial | S1 hushed: -8.6 / -9.5 dB vs control | S2, S3: +0.1..+1.1 dB vs control, normal voicing |
| C | `[quietly]` mid-text | hushed: -5.2 dB | next sentence normal: -0.5 dB |

Reading: a delivery tag's effect lasts about ONE sentence and does not survive a `[long pause]` x2 stack, text-initial or mid-text. (B was part of the set and is consistent with this reading; its numbers are not reproduced here.) Not measured on v3 or other voices.

Practical rules:
- Put the tag on EACH sentence or paragraph it must affect; never assume a register carries across a pause stack, paragraph or chunk. A sustained register re-applies one hold tag at every paragraph and chunk start and every ~2 sentences / ~250 characters (`elevenlabs-voice-director-sleep`).
- Expect level steps where an untagged paragraph follows hushed text. [measured, one v4 voice, single take] an untagged paragraph after hushed text measured +1.8 dB over the preceding paragraph (per-paragraph LUFS on the rendered take).
- Fix order for such a step: measure per-paragraph loudness -> level-match in post -> use a voiced-calm tag only if delivery, not level, is the problem (`[gently]` is [measured, single take], below; other voiced-calm tags [unverified]).

### `[gently]` as a voiced-calm cue, one v4 voice (2026-10-19)

[measured, single take] One voice, `eleven_v4` over OpenRouter, an intro passage whose untagged notice sentence stood above its neighbours. Variant B is the same text with only `[gently] ` inserted before the notice sentence; one take each, no re-rolls; instrument measurements plus a transcript check.

| Take | Notice vs preceding paragraph | Tag spoken | Notice voiced fraction |
|---|---|---|---|
| Untagged | +1.80 dB | n/a | not recorded here |
| Variant B (`[gently]` on the notice) | -0.07 dB | no | 0.56 |

Reading: `[gently]` on this v4 voice behaved as a voiced-calm cue. On a sentence it flattened that sentence to its neighbours' level (step -0.07 dB vs +1.80 dB untagged), was not spoken aloud, and stayed voiced (voiced fraction 0.56, not a whisper; v4 `[quietly]` is 0.05-0.14). It slowed the sentence slightly (6.83 s) and the whole take measured quieter (-26.44 LUFS). It resolved the level step in that test. Single take, one voice, one position: audition before relying on it elsewhere; the scope rule above (one sentence) applies. Not measured on v3 or other voices.

### Model-aware use of these numbers (sleep / quiet narration, timing, switching)

[measured-controlled] on one voice (`sarah`) and one sentence; measurements are not listening judgements; several results are one take. Everything not stated here is [unverified].

- Quiet narration (e.g. a sleep podcast): a whole episode is directed with `elevenlabs-voice-director-sleep`. On a single sentence on v4, `[whisper]`, `[quietly]` and `[softly]` measured mostly unvoiced/breathy (~5-14% voiced, 36-72% of energy above 4 kHz); on v3 the same tags stayed voiced and quieter (flag 2026-10-19: on a second voice in-passage, v3 `[quietly]`/`[whispers]` made no measurable change; see the quiet probe). For a sustained v4 sleep read, use one project hold tag (`voice.sleep.hold_tag`, e.g. `[quietly]` or `[gently]`) re-applied as below, keep `[whisper]` / `[softly]` out of the read (closing lines only if the project lists them in `voice.sleep.endings_tags`, default none), and label other calm tags (`[calm]`) [unverified] until auditioned; plain text with calm punctuation (short sentences, no CAPS) is the no-tag A/B arm. Use post-production level, not a volume tag, to hit a loudness target.
- Cue placement: a delivery tag lasts about one sentence (scope section above), so a single cue at a chunk head or cold open does not hold a register. A sustained register puts the hold tag at every paragraph start and every chunk start (an identical opening string per chunk) and re-applies it every ~2 sentences / ~250 characters inside long paragraphs ([measured on one v4 voice, single segment, 2026-10]: re-tagging removed a mid-paragraph loudness and pitch jump; verify on your voice). Project decides: `voice.sleep.hold_tag`, `voice.sleep.retag_every_sentences`, `voice.sleep.retag_every_chars`.
- Timing: `[long pause]` is not a fixed duration (4.78 s on v3, 1.78 s on v4, one take each; a second voice's v4 x2 1.3-2.4 s across identical takes). Never assume pause lengths across models; a timing doctrine tuned on one model must be re-auditioned on the other.
- v4 rests: designed rests longer than a sentence break become split points with the rest inserted in assembly (room tone or digital silence, per the project's rest fill) (**Pause rests, splits and the split map**); doubled pause tags overshoot and drift on v4. A project records its own timing doctrine in its project file (`timing_doctrine`). [measured, one voice, single takes] Runtime compresses on v4: the same intro text ran ~94 s on v3 and ~66 s on v4. Any post-production rate taper is an `audio-treatment` step ("Rate taper and chain order").
- Level: untagged v4 measured ~8 dB quieter than v3. Re-normalize to the project's target (the project owns it, e.g. -19 LUFS for sleep); never rely on raw render level.
- Model-switch checklist: (1) regenerate tagged text for the new model's profile; (2) re-audition every tagged line and pause; (3) re-measure loudness and re-normalize to the project target; (4) transcript audit for spoken tag text, dropped words, unrequested v4 non-verbals; (5) record the render model per take; (6) user listening gate again. Procedure owners: `elevenlabs-voice-director` (profiles, A/B audition recipe) and `speech-render` (migration section). Do not mutate an existing project's profile, `timing_doctrine` or loudness automatically.

### Open items (2026-10-19)

Not established; do not present as rules:
- [unverified] `[calm]` / `[slowly]` as voiced-calm tags: may shift voicing, be ignored, or be spoken. (`[gently]` is [measured, single take] on one v4 voice: see "`[gently]` as a voiced-calm cue".)
- [unverified] Whether OpenRouter's `eleven-v4` route passes a speed field in practice; even if present it is constant per call (no taper).
- [inferred] `max_chunk_chars: 4000` for v4; v3's ~2,700 is a v3-era observation, not a v4 rule.
- [unverified] Whether v3's under-delivery still justifies doubled pause tags on v3 (untested with v4-era data).
- Name pronunciations and respellings are project decisions; record them in the project, not here.

### Test recipe (reproduce or re-run on your own voice/model)

- Route: OpenRouter `POST /api/v1/audio/speech`; models `elevenlabs/eleven-v3` and `elevenlabs/eleven-v4`; `voice: sarah`; `response_format: mp3`; no `instructions`, no `speed`, no `provider.options` (default settings), identical for both models; rendered 2026-10-18.
- Base text: `The light is almost gone.` Cases: no tag (control, 2 takes), `[whisper] ` (2), `[quietly] ` (2), and one take each of `[softly]`, `[shouts]`, `[excited]`, `[sad]`, `[laughs]`, `[sighs]` placed before the sentence, plus `The light is almost gone. [long pause] Now it is dark.`
- Measurement: ffmpeg decode to 16 kHz mono; `ebur128` for integrated LUFS and true peak; pure-Python (numpy-free) for the rest. Active audio = 20 ms frames within 40 dB of the loudest frame; RMS, zero-crossing rate, spectral centroid and the >4 kHz share are over active frames (band limited to 0-8 kHz by the 16 kHz decode); voiced = 32 ms frames with normalised autocorrelation peak above 0.5 between 70 and 400 Hz (crude; F0 is unreliable when few frames are voiced, so F0 is not tabulated); the pause is the longest silent run between active frames.
- Before final renders, re-run a few of your own tagged lines on your actual voice and model (two takes each, so you can see run-to-run noise) and compare against an untagged control rendered on the same model; these numbers are one voice, one sentence.

## Limits and chunk continuity

| Model / endpoint | Request ceiling | Evidence / scope |
|---|---|---|
| v4 | 10,000 characters; 90+ languages | [verified] [ElevenLabs models](https://elevenlabs.io/docs/overview/models); also OpenRouter announcement. |
| v3 | 5,000 characters; 70+ languages | [verified] Models page; OpenRouter announcement calls it v3 Conversational. |
| v4 Turbo | 10,000 characters on OpenRouter | [verified] [OpenRouter announcement](https://openrouter.ai/blog/announcements/elevenlabs-on-openrouter); direct-API limit [unverified], absent from the supplied models page. |
| Multilingual v2 | 10,000 characters on OpenRouter | [verified] OpenRouter announcement. |
| Flash v2.5 / Turbo v2.5 | 40,000 characters on OpenRouter | [verified] OpenRouter announcement. |
| Flash v2 / Turbo v2 | 30,000 characters on OpenRouter | [verified] OpenRouter announcement. |
| Text to Dialogue | 2,000 characters total across `inputs[].text` | [verified] Each turn has its own `voice_id`, tags inside that turn. [Text to Dialogue](https://elevenlabs.io/docs/overview/capabilities/text-to-dialogue). Do not apply the TTS ceiling to this endpoint. |

Repository chunk policy: respect `audio.chunking.target_chars` and `audio.max_chunk_chars`, bounded by the selected route/endpoint ceiling. Split at paragraph or sentence breaks, never inside a sentence or bracket tag. Count the final outgoing text including tags and any repeated tag. An oversized sentence needs an explicit edit decision; do not silently split it or exceed the ceiling.

- [observed] v3's ~2,700-character conservative target comes from a production runbook (unreliable finalization near 5,000); it is v3-specific and not the vendor hard limit. Do not carry it to v4.
- [inferred] v4/v4 Turbo `max_chunk_chars: 4000` in the model lookup is an untested repository choice, derived well below OpenRouter's 10,000 ceiling. Confirm the direct v4 Turbo ceiling first. This value does not select a model or set a project default. The sleep register uses its own band (`voice.sleep.chunk_chars`, `elevenlabs-voice-director-sleep`).
- [inferred] Repeat the active **emotion** tag at each chunk start for continuity; do not repeat a one-off laugh, gasp or SFX as if it were a persistent emotion. **Delivery** tags are different: a chunk-head delivery tag covers about one sentence, so tag each sentence that needs it; a sustained register opens every chunk with its hold tag (identical opening string) and re-applies it inside long paragraphs (cue placement above).
- Rests longer than a sentence break on v4: see **Pause rests, splits and the split map** below.
- [verified] Direct TTS API supports `previous_text` / `next_text`. OpenRouter forwards them under `provider.options.elevenlabs`; its announcement uses the last ~500 characters of the previous chunk and first ~500 of the next. Pass these where the selected route exposes them; do not invent MCP/web fields. Sources: [TTS convert API](https://elevenlabs.io/docs/api-reference/text-to-speech/convert), [OpenRouter announcement](https://openrouter.ai/blog/announcements/elevenlabs-on-openrouter).

## Pause rests, splits and the split map

Design note: there is no separate assembly skill; the split map is the handoff artifact between direction and assembly.

**Hard rule (v4).** A designed rest longer than a sentence break MUST be expressed as a split at that boundary plus an entry in the split map; assembly inserts `owed = max(0, target − measured native quiet)` at the join, filled with room tone or digital silence per the project's rest fill (room tone is optional). No pause tag for that rest, and never doubled pause tags. Basis: [measured on one v4 voice, one long-form episode, single takes, 2026-10] `[long pause]` x2 delivered a median ~2.5 s against the ~1.6 s short-probe estimate (**Measured pause ladder** above), about half of them above the probe's maximum; a single `[long pause]` gave ~2.3-2.8 s against ~1.1 s; the same x2 tag lengthened across the episode (~1.4 s early to ~2.8 s late); every assembly-inserted rest landed within ~1% of target. Tag and delivery effects also do not accumulate across stacks or boundaries (**Delivery-tag scope and reset** above). Verify on your voice. This supersedes the 2026-10-19 rule "tags up to `[long pause]` x2 (~1.6 s ceiling), then split".

**v3 (v3-specific).** The v3-era practice timed rests with tags (probe: x2 ~3.0 s; x3 no gain, one take); never stack more than 2. Whether v3 under-delivery still justifies doubling is untested with v4-era data; the split method works on v3 too [inferred].

**Project decides**: whether assembly may insert silence at all (`voice.sleep.post_silence`; some personas forbid splicing silence in post, which caps long rests at what an untagged paragraph break delivers) and the rest length that forces a split (`voice.sleep.split_min_rest_s`). The project's `timing_doctrine` records the choice.

- **Split map**: every direction job emits one beside the tagged chunks (`<episode>-splitmap.json`; fields and example in `elevenlabs-voice-director` -> Step 6 "Split map"): per boundary, its id, location, target seconds, any tag (null on v4) and the assembly silence owed (target minus measured edge silence, computed at assembly).
- **Assembly** consumes the map: joins on its boundaries, owed silence filled per the project's rest fill (room tone crossfaded, or digital silence), relative levels kept, silence-detect QC against the map. See `audio-treatment` -> "Split-map assembly".
- **Tradeoff**: more splits = exact pause control, more seams; longer chunks = smoother flow, in-chunk rests left to the voice's own phrasing. Splits belong at scene/paragraph boundaries (held beats), never mid-paragraph, unless the project opts in to `audio.chunking.split_at` (sentence or marked-hold split points: `elevenlabs-voice-director` → Step 6 "Split granularity"). Length-driven chunk breaks still follow the chunk policy above; they appear in the map too.

## Settings and pitfalls

[verified] Voice-setting defaults: `stability: 0.5`, `similarity_boost: 0.75`, `style: 0`, `use_speaker_boost: true`. Lower stability broadens emotional range; higher values can sound monotonous. [inferred] Lowering stability for expressive tag work is practitioner guidance, not a guaranteed model-specific recipe. [unverified] Creative/Natural/Robust stability-mode behavior and differences on v3 vs v4 are not established here; do not prescribe modes. Source: [best practices](https://elevenlabs.io/docs/overview/capabilities/text-to-speech/best-practices).

[verified] `apply_text_normalization` defaults to `auto`; `language_code` uses ISO 639-1. Generation is nondeterministic: compare several authorized takes and use `seed` where exposed, without promising identical output. Source: [TTS convert API](https://elevenlabs.io/docs/api-reference/text-to-speech/convert).

If tags are spoken, check the selected upstream model (`eleven_v3`, `eleven_v4`, `eleven_v4_turbo`) and square-bracket syntax before adding more tags. [inferred] Unsupported models or malformed brackets can cause literal readings. Overuse/clashing tags may blur performance. Listen for unrequested v4 non-verbals. Keep consent, AI-disclosure, listening gates and per-take spending authorization unchanged.

## OpenRouter ElevenLabs TTS

[verified] `POST /api/v1/audio/speech` accepts bracket tag syntax in `input`. This is syntax support only; see model-specific delivery and listening review above. Tags count toward the character ceiling and billing, per Unicode code point. `voice` is one of 21 named voices, e.g. `george`, `sarah`, `adam`; confirm the selected model's voices. `response_format` is `mp3|pcm`, default `pcm`. Failed generations are not billed. Sources: [OpenRouter TTS](https://openrouter.ai/docs/features/multimodal/tts), [ElevenLabs announcement](https://openrouter.ai/blog/announcements/elevenlabs-on-openrouter). The [v4 model page](https://openrouter.ai/elevenlabs/eleven-v4) states no character limit; the announcement supplies 10,000.

```json
{"model":"elevenlabs/eleven-v4","input":"[whispers] Stay close. [excited] We made it!","voice":"sarah","response_format":"mp3","provider":{"options":{"elevenlabs":{"apply_text_normalization":"auto"}}}}
```

[verified] Put ALL direction in `input` as tags/punctuation. Omit `instructions`: the TTS docs say it is ignored except by OpenAI gpt-4o-mini-tts, Gemini TTS and Fish Audio.

[observed] instructions: accepted without error but had no observable effect on elevenlabs/eleven-v4 in one A/B (2026-10-17); do not rely on it; output is nondeterministic so one test is indicative, not proof

Maintainer-reported A/B: `elevenlabs/eleven-v4`, voice `sarah`, `mp3`; input `[whispers] Creator Studio is live. [excited] This is a test of the voice.` With vs without `instructions="Speak very slowly in a deep angry growl."`: both HTTP 200, durations 4.16 s with vs 4.40 s without, no slow/growl effect. This is one observation, not general experimental proof.

[verified] v4 and v4 Turbo have no speed control and reject a non-default speed on OpenRouter. Omit `speed` entirely for these models on either route; use pacing tags/punctuation. The announcement documents speed 0.7-1.2 only for Multilingual v2 and Flash, under `provider.options.elevenlabs.voice_settings.speed`. Speed support on v3/Turbo v2 family is [unverified] here; omit it unless established for the selected route. Flag 2026-10-19: in project use it is still [unverified] whether the `eleven-v4` route passes a speed field at all; if it does, it is constant per call and cannot make a rate taper (post: `audio-treatment`).

[verified] ElevenLabs-specific keys go under `provider.options.elevenlabs`:

| Keys | Rule |
|---|---|
| `seed`, `previous_text`, `next_text` | Seed where supported; continuity context ~500 characters each way. |
| `language_code`, `apply_text_normalization`, `apply_language_text_normalization` | ISO 639-1 language; normalization defaults to `auto`. |
| `pronunciation_dictionary_locators` | Accepted parameter; v3/v4 behavior [unverified]. |
| `voice_settings` | `stability`, `similarity_boost`, `style`, `use_speaker_boost`, `speed` only where supported; no speed for v4/v4 Turbo. |
| `output_format` | Must match `response_format` codec, e.g. `mp3_22050_32` with mp3, `pcm_16000` with pcm. |

[verified] Unlisted keys are ignored; a listed key with an invalid value returns 400. The TTS docs page has no ElevenLabs section: the [announcement](https://openrouter.ai/blog/announcements/elevenlabs-on-openrouter) is the source for these key names and speed rules. Direct API support does not establish MCP/web controls. [unverified] Direct-account voice-id portability to OpenRouter and connector/UI exposure of context or v4 Turbo must be checked before use.
