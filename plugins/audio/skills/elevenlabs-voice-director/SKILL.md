---
name: "elevenlabs-voice-director"
description: "Direct an ElevenLabs read: analyze a script for emotion, subtext and pacing, then emit the voice profile, settings and ready-to-render text (audio tags, punctuation, normalization, pronunciation) for the model the project voice profile pins (v4, v3 or v2 family). Also casts one voice per speaker for dialogue. Directs only; rendering is speech-render (paid, approval first)."
---

# ElevenLabs Voice Director

You are a voice director. Given a script, you (1) analyze it the way an actor and director would, then (2) translate that performance into the exact text ElevenLabs needs, so the generated audio sounds acted, not read.

The core idea: in ElevenLabs the "prompt" IS the spoken text. There is no scene description field. Every bit of direction has to live inside the text itself: audio tags, punctuation, capitalization, line breaks, word formatting, plus the voice and a few settings.

Show the analysis visually (arc sparkline, table, color-coded beats) before the final prompt.

**In Creator Studio** this skill is the director, not the renderer: the model, voice id and settings come from the
project's voice profile (`project.json` voices[]; resolve explicit production choices into the profile for their scope), the render text it writes goes to `speech-render`, and
every render is a paid step covered by existing authorization and project spending/review rules recorded by `studio`. Siblings: `ai-voiceover` (script for the ear,
consent + disclosure gate), `speech-render` (render), `audio-post-mix` / `audio-master` (mix, master). The original
standalone `elevenlabs-voice-director` skill is fully contained here.

---

## OpenRouter voice choice
For route `openrouter`, resolve `models.voice` (`default` / `allowed` / `locked`) and the exact voice profile under the same production/project rules. Read `studio` → `references/models/openrouter.json` and `references/openrouter.md`. Use the chosen model’s live `supported_voices`, not `creative_list_voices`: account/library ids from direct ElevenLabs may not be usable, and the voice may differ. Explain the conflict rather than silently swapping a kept voice. For ElevenLabs models on OpenRouter, the tag direction below still applies; route-specific voices, settings and preflight come from those OpenRouter references. Do not use direct voice-library lookup or MCP preflight on this route. Translate direction only into controls the selected model supports (non-ElevenLabs models need model-specific guidance). Render through `speech-render` with OpenRouter pricing approval, not MCP `estimate_only`. OpenRouter does not provide ElevenLabs music, SFX or voice-library management.

## Workflow

### Step 1 - Intake (ask only what you can't infer)
Infer from the script whenever possible. Ask at most one short question set if these are unknown and matter:
- **Use case**: ad, narration/audiobook, YouTube/explainer, character/drama, meditation, podcast, game, agent/IVR.
- **Model**: always the project voice profile's `model`; resolve explicit production choices into that profile for their scope. If absent, obtain a choice before rendering; no code default. Recommend v4 for new work, keep selected v3. See Step 3. **The model is an input to this director, not a detail applied afterwards**: read it before choosing a single tag, and write tags and wording FOR THAT MODEL (4a-1 tag profiles). Also read the project's register and timing rules (e.g. a sleep/quiet-narration register, `timing_doctrine`, loudness target) from the project profile; they are project-owned and this skill never edits them.
- **Voice**: the project's locked voice (voices[]) if there is one; else do they have one (library voice, IVC,
  PVC)? If not, you'll recommend a voice profile (`direct_voice`) and real picks from `creative_list_voices`.
- **Speakers**: one or several?
- **Target length / pace** if it's timed (e.g. a 30s ad).
If the user is absent or says "just do it", state reasonable creative assumptions at the top; do not invent a missing voice/model selection.

### Step 2 - Script analysis (the director's read)
**If `emotional-director` ran** (the script carries `@ed` lines, or an emotion-direction sidecar exists), do not re-derive the emotional read: take its passages as your beat map (emotion, intensity, energy, pace, role, key word, `land`), keep to its foundation line, and spend this step on what is yours: hazards, non-verbal candidates the foundation allows, and the model-specific translation in Steps 3-6. Strip the `@ed` lines before anything is rendered; the spoken words are unchanged by that stage. If the model or the project's approved tag list cannot express a value, say so; never raise or lower the direction to fit. If it did not run, do the read below as before.

Break the script into **beats**: a beat is a stretch where the speaker's intention or emotion stays the same. A new beat starts when the emotion, intention, energy or listener changes. Typical script: 4-12 beats.

For each beat determine:
| Field | What to decide |
|---|---|
| Beat # + first words | So the user can find it |
| Intention | What the speaker WANTS from the listener (reassure, provoke, seduce, warn, sell, confess) |
| Emotion | Primary emotion + any secondary (e.g. "relief, under it guilt") |
| Subtext | What's meant but not said; this is what makes it sound human |
| Energy 1-5 | 1 = whisper/intimate, 3 = conversational, 5 = shouting/peak |
| Pace | slow / natural / brisk, and where pauses land |
| Key word(s) | The one or two words that carry the meaning (candidates for CAPS) |
| Delivery | The concrete vocal choice: softly, breathless, deadpan, warm smile in the voice, trembling, etc. |

Then identify:
- **The arc**: how energy and emotion travel from first line to last. Find the **turn** (biggest emotional shift) and the **peak** (highest energy). Every good read has contrast; if the whole thing is a 4, it will sound flat. Build valleys before peaks.
- **Non-verbal moments**: places a real person would sigh, laugh, gasp, swallow, clear their throat, catch their breath. Only where it's earned by the text.
- **Hazards**: numbers, dates, URLs, acronyms, names, foreign/technical words, stage directions written in the script, parentheticals that shouldn't be spoken.

### Step 3 - Choose the model
The model ALWAYS comes from `project.json` `voices[].model`, never a code/provider default. Resolve explicit production choices into the voice profile for their scope; `models.voice` constraints still apply. v4 is ElevenLabs' recommendation for new work; v3 remains valid when selected by the project/voice. Recommend a choice if the profile is missing, then record the resolved profile before rendering.

Read the `studio` skill → `references/models/elevenlabs.json` (`tags_supported`, request ceilings, conservative chunk targets) and `references/elevenlabs-tagging.md` (cited evidence, route controls and caveats). Evidence labels below follow that reference.

| Model | Direction tools |
|---|---|
| `eleven_v4` / `eleven_v4_turbo` | Bracket tags, punctuation, CAPS; no SSML `<break>` and no `speed`. Confirm Turbo availability/limit on the selected direct route. |
| `eleven_v3` | Same bracket syntax; keep the selected voice/model. No `<break>`. PVCs not fully optimized. |
| Multilingual v2 / Flash / Turbo v2 and v2.5 | No audio tags; strip/convert them to punctuation and supported settings. Older-model `<break>` up to 3s. |

Map the resolved model to a tag profile before Step 4: `eleven_v3` -> profile `v3`; `eleven_v4` / `eleven_v4_turbo` -> profile `v4` (Turbo is [unverified] against the measured set, which used `eleven-v3` and `eleven-v4` only; audition it); non-tag models -> none. The model is read from the project, never chosen here to suit the tags.

### Step 4 - Write the direction into the text

#### 4a. Audio tags (check `tags_supported` first)
[verified] v3/v4 share free-form natural-language tag syntax. Delivery is model-specific ([maintainer-observed]); follow `studio` → `references/elevenlabs-tagging.md` → **Model changes and listening review** when switching or mixing models. A model switch on an approved production requires the user’s listening gate again; never record an approval on the user’s behalf. Use square brackets immediately before the affected words, never parentheses/braces. Emotion carries across the line; add a new tag when delivery shifts. Combine qualities in one bracket: `[whispering, fearful] Stay close.`

**Delivery-tag scope ([measured on one v4 voice, single takes, 2026-10]; rules in 4a-5; verify on your voice):** a delivery tag such as `[quietly]` lasts about ONE sentence and does not survive a pause-tag stack, whether it is text-initial or mid-text. Put the tag on each sentence it must affect (or, for a sustained register, re-apply a hold tag, 4a-2); never assume a register carries across a paragraph, chunk or pause stack.

| Family / example | v3 | v4 (including Turbo family) |
|---|---|---|
| Delivery: `[whispers]`, `[shouts]`, `[softly]` | Match the voice's natural range. | Follows unfamiliar delivery more reliably; results may still be suboptimal. |
| Emotion: `[curious]`, `[excited]`, `[crying]` | Carry across the line. | Same syntax and placement. |
| Non-verbal: `[laughs]`, `[sighs]`, `[gasps]` | Use where earned. | Listen for unprompted stammers/groans too. |
| Pace: `[pause]`, `[long pause]`, `[slowly]`, `[rushed]` | Less deterministic than SSML. | Use tags/punctuation for pace; omit speed. |
| SFX: `[applause]`, `[gunshot]` | Less consistent; may be misread. | Same caution; describe vocal quality explicitly (`[low, gravelly voice]`). |
| Accent / character / `[sings]` | Experimental; test per voice. | Experimental; test per voice. |

[measured-controlled] Model differences (one voice, one sentence; two takes for `[whisper]`/`[quietly]`, one for the rest; table and noise in the shared reference's **Measured v3 vs v4**):
- `[whisper]`, `[quietly]`, `[softly]` differ above noise: on v4 the measurements are consistent with a much breathier, unvoiced delivery; on v3 they are consistent with a quieter but still voiced one. Pick the tag per model and audition it.
- `[shouts]` was much louder on v4 only (indicative, 1 take); `[excited]` marginal; `[sad]`, `[laughs]`, `[sighs]` showed no model difference above noise.
- Repeat a delivery tag at each sentence where it must persist (an [maintainer-observed] whisper did not hold past the first sentence on either model); a section-level tag covers only about its first sentence on v4 (4a-5).
- A v3/v4 switch changes level (v4 control ~8 dB quieter): loudness-check and level-match after switching, before comparing or mixing.
- Do not rely on `[long pause]` for a fixed length: measured gaps were ~4.8 s (v3) and ~1.8 s (v4) for the same text. On v4, a designed rest longer than a sentence break is a split point with the rest inserted in assembly (room tone or digital silence, per project), not a tag (4a-3).
- Re-run a few tagged lines on your own voice and model before final renders.

#### 4a-1. Tag profiles: choose the profile from the model
Pick the profile that matches the project's resolved model, use only it for the whole script, and record it (Step 7 output). Never carry a script tuned on one model to the other unchanged. Everything below is from one voice (`sarah`), one short sentence, mostly one or two takes; measurements are signal properties, not listening judgements. Treat the profiles as starting points to audition on the project's own voice (4f).

| | Profile `v3` (`eleven_v3`) | Profile `v4` (`eleven_v4`, `eleven_v4_turbo`) |
|---|---|---|
| Quiet tags `[whisper]` `[quietly]` `[softly]` | [measured-controlled] Quieter, still voiced (voiced fraction moved 2-14 points, spectrum unchanged). Usable for a quiet read; audition level and intimacy. [measured on a second voice, in-passage, single takes, 2026-10] `[quietly]` and `[whispers]` showed NO measurable level change on v3 (consistent with being ignored); "quieter" is voice/position dependent, not a v3 rule. | [measured-controlled] Mostly unvoiced and breathy: ~5-14% voiced, 36-72% of energy above 4 kHz. [measured on a second voice, single takes, 2026-10] v4 `[quietly]`: ~9-12 dB below control, voiced fraction 0.05-0.14 (vs ~0.5-0.7 untagged), >4 kHz share 34-43% (vs ~10%): on its own sentence it acts as a whisper; `[whispers]` adds nothing beyond it. Never `[whisper]` / `[softly]` as a body tag for sustained narration. A sustained quiet register uses one hold tag re-applied through the read (4a-2), auditioned. |
| Pause `[pause]` / `[long pause]` | [measured-controlled] `[long pause]` gap ~4.8 s on the test sentence (1 take). Not fixed. [measured on a second voice, single takes] x1 ~2.3 s, x2 ~3.0 s, x3 ~2.5 s (stacking does not scale). v3-era practice timed rests with this ladder; audition it. | [measured-controlled] ~1.8 s on the same sentence (1 take). Not fixed. [measured on a second voice, short probe] singles ~1.0-1.1 s, x2 ~1.6 s; in a long-form render the same tags overshot and drifted (4a-3). Not a timing tool on v4: designed rests become splits. A tuned v3 pause map is not valid on v4. |
| Emphasis tags `[shouts]` `[excited]` | [measured-controlled] ~no change in level vs untagged (indicative, 1 take). | [measured-controlled] `[shouts]` ~+8 dB over its own untagged control (1 take); `[excited]` marginal. Mind level jumps. |
| `[sad]` `[laughs]` `[sighs]` | No model difference above noise; `[laughs]`/`[sighs]` add ~0.6-0.8 s. | Same. Also [verified] v4 may add unprompted stammers/groans: listen for them. |
| Level (untagged) | [measured-controlled] Louder: -13.2 LUFS, peak -1.0 dBFS. | [measured-controlled] ~8 dB quieter: -21.3 LUFS, peak -8.4 dBFS. [measured] A second voice was also ~8 dB quieter on v4 than on v3. |
| Tag following | [verified] Same syntax; PVCs not fully optimized. | [verified] Follows unfamiliar/off-voice direction more reliably, but results may still be suboptimal. |
| Not measured | [unverified] Anything beyond the tags above: other tags, stability/similarity settings, other voices, long passages, other languages. | Same. |

Rules:
- Persistence is not assumed on either model: [maintainer-observed] a whisper did not hold past the first sentence; [measured on one v4 voice] a delivery tag lasts about one sentence and resets across a pause-tag stack (4a-5). Repeat a delivery tag on each sentence where it must hold, then audition. Scope on v3 is not measured.
- Never copy a tag from one profile into the other because it "worked before". If the model changes, regenerate the tagged text from the plain script with the new profile and treat it as a new draft.
- Tags not in this table are [unverified] on both models until auditioned; label them so in the output.

#### 4a-2. Quiet / sleep / wind-down narration
**A whole sleep, bedtime or meditation episode is directed with `elevenlabs-voice-director-sleep`**, which owns that register's tag vocabulary, hold-tag placement, pause rules, chunking and QC. This section is the model background it builds on, and the guidance for a quiet moment inside another register.
Goal: slow, soothing, close, level. First read the project's register and banned moves (e.g. no whisper, intensity floor 1-2, `timing_doctrine`, loudness target such as -19 LUFS); they outrank anything below.
- **On v3 (profile `v3`)**: [measured-controlled] the quiet tags stay voiced and only get quieter, so `[softly]` / `[softly, slowly]` once at the top, repeated at section starts, is the v3-era starting point. [maintainer-observed] v3 native-tag timing is where a project's timing doctrine usually lived; follow that doctrine and audition every pause (4a-3).
  Caveats: [measured on a second voice, in-passage, single takes, 2026-10] v3 `[quietly]` / `[whispers]` made no measurable level change, so a v3 quiet tag may do nothing on another voice (`[softly]` was not in that set). Delivery-tag scope on v3 is unmeasured; on v4 it is about one sentence (4a-5), so section-start-only placement is v3-specific. Audition before relying on it.
- **On v4 (profile `v4`)**: [measured-controlled] `[whisper]`, `[quietly]` and `[softly]` rendered mostly unvoiced/breathy on a single sentence (~5-14% voiced; 36-72% of energy above 4 kHz). [measured on a second voice, single takes, 2026-10] v4 `[quietly]` dropped ~9-12 dB vs its control and turned breathy/unvoiced (voiced fraction 0.05-0.14 vs ~0.5-0.7 untagged; >4 kHz share 34-43% vs ~10%): on its own sentence it acts as a whisper, and `[whispers]` adds nothing beyond it. [measured on one v4 voice, one long-form episode, single takes, 2026-10] used as a paragraph-start hold tag through a full sleep read, `[quietly]` measured as a calmer, still-voiced, usable read. Whether a hold tag at every paragraph reads as a whisper chain over a long listen is a listening question; audition it. `[whisper]` / `[softly]` stay out of the body (in a sleep episode, closing lines only if the project lists them in `voice.sleep.endings_tags`, default none).
- **Cue placement**: a delivery tag lasts about one sentence (4a-5), so one cue at a chunk head, section head or cold open does not set a register. For a sustained quiet register, put the hold tag at **every paragraph start and every chunk start** (an identical opening string per chunk) and **re-apply it every ~2 sentences / ~250 characters** inside long paragraphs ([measured on one v4 voice, single segment, 2026-10]: re-tagging removed a mid-paragraph loudness and pitch jump; verify on your voice). Project decides: `voice.sleep.hold_tag`, `voice.sleep.retag_every_sentences`, `voice.sleep.retag_every_chars` (sleep variant 4a-1). For a single quiet moment in another register, tag only that sentence.
- **v4 alternatives to try (all [unverified] unless marked; each must be auditioned on the project voice and labelled `unverified` in the output)**:
  1. No quiet tag at all: plain text with calm structure: short sentences, ellipses for weight, commas for breath, line breaks for gravity, no CAPS, no exclamation marks (punctuation direction is [verified] to work as a signal; its effect on a sleep read is not measured).
     [maintainer-observed, 2026-10, one excerpt, one take, one stock v4 voice] a no-tag version (`[quietly]` removed, pause tags turned into `...`) felt more natural than the same text with `[quietly]` + `[long pause]`, and the v4 `[quietly]` take was heard as a true whisper, more so than on v3. One excerpt, one take: promote to a default only after the A/B audition (4f) on your own voice.
  2. A voiced-calm tag instead of a volume tag. `[gently]` is [measured, single take, one v4 voice, 2026-10] (4a-6): not spoken, stays voiced (voiced fraction 0.56), flattens its sentence to its neighbours' level. `[calm]`, `[gentle]` and `[slowly]` remain [unverified] (may shift voicing, may be ignored, may be spoken).
  3. Voice choice and settings (a naturally low, soft voice; stability) do more of the work than tags [inferred]; do not prescribe stability modes ([unverified]).
  4. Reduce level in post, not with tags: record at natural voicing and normalize/attenuate to the project target (`audio-master`).
- Measure what you can: voiced fraction and the share of energy above 4 kHz are cheap to compute (the shared reference's test recipe); a high-frequency-heavy, low-voicing take is a flag to listen to, not a verdict.
- Never use the quiet-tag outcome as a substitute for the project's loudness target: tags change delivery, the target is met in post.

#### 4a-3. Timing across models
- Do not assume any pause length. [measured-controlled] `[long pause]` was ~4.8 s on v3 and ~1.8 s on v4 for the same sentence (1 take each); neither is a fixed duration, and run-to-run variation is expected. [measured on one v4 voice, 2026-10] the same `[long pause]` x2 delivered 1.3-2.4 s across identical takes (~0.5 s take-to-take spread): write pause targets as ranges, not values.
- **Hard rule (v4): a designed rest longer than a sentence break is a split point, not a tag.** Write it as a split at that boundary plus a split-map entry (Step 6, "Split map"); assembly inserts `owed = max(0, target − measured native quiet)` at the join, filled with room tone or digital silence per the project's rest fill (`audio-treatment` -> "Split-map assembly"). No `[pause]` / `[long pause]` for that rest, and **never doubled pause tags**. Basis: [measured on one v4 voice, one long-form episode, single takes, 2026-10] `[long pause]` x2 delivered a median ~2.5 s against a ~1.6 s short-probe estimate, about half of them above the probe's maximum; a single `[long pause]` gave ~2.3-2.8 s against ~1.1 s; and the same x2 tag lengthened across the episode (~1.4 s early to ~2.8 s late). Every assembly-inserted rest landed on target. Verify on your voice. This supersedes the earlier "tags up to `[long pause]` x2 (~1.6 s ceiling), then split" rule: on v4 the measured problem is over-delivery and drift, not a ceiling.
- **v3 (v3-specific)**: the v3-era practice timed rests with tags ([measured on a second voice, single takes] x1 ~2.3 s, x2 ~3.0 s, x3 no gain: never stack more than 2). Whether v3's under-delivery still justifies doubling is untested with v4-era data; the split method works on v3 too [inferred]. Audition before keeping a v3 tag-timed pause map.
- Short and expressive pauses stay in the text: full stops, commas and line structure are the voice's own, steadier phrasing; a `[pause]` / `[short pause]` before a reveal or punchline (Genre recipes) is an expressive beat, never a timing promise.
- Project decides: whether assembly may insert silence at all (`voice.sleep.post_silence`; some personas forbid splicing silence in post: then long rests are capped at what an untagged paragraph break delivers) and the rest length that forces a split (`voice.sleep.split_min_rest_s`). Ask when unset; never resolve it here.
- [measured, one voice, single takes] Runtime compresses on v4: the same intro text ran ~94 s on v3 and ~66 s on v4.
- A post-production rate taper (slowing the story read over time) is a mix/treatment step, not a tag: see `audio-treatment` -> "Rate taper and chain order".
- Pauses that matter (a sleep podcast's breath between paragraphs) are taken out of the tag system: separate chunks plus silence in assembly (`audio-post-mix` / `audio-treatment`), measured afterwards. Ellipses and line breaks are for short pauses only; a designed long rest follows the hard rule.
- When the project has a `timing_doctrine`, apply it as the project states it, record which model it was tuned on, and flag it `re-audition` if the render model differs. Do not rewrite the doctrine yourself.
- Model-switch checklist (also in the shared reference): (1) regenerate tagged text with the new profile; (2) re-audition every tagged line and every pause; (3) re-measure loudness and re-normalize to the project LUFS target (never trust raw level: untagged v4 ~8 dB quieter than v3 in the measured set); (4) transcript audit: transcribe the render and confirm no tag text was spoken and no words were dropped; (5) record the model per take; (6) the user's listening gate runs again.

[verified] Test against the voice: even v4 may not make a serious voice sound convincingly playful. [inferred] Direct audible qualities, not body motion; tag beat changes sparingly and avoid clashing directions. “1-3 tags per sentence” is practitioner guidance, not an official quota. [unverified] English-only tags are a convention, with no official localization guidance supplied. Detailed examples and sources live in the shared reference above.

#### 4a-4. Short-probe pause ladder and quiet-tag behaviour (second voice, 2026-10, [measured])
One take per condition on a short in-passage probe (one library voice, v3 vs v4 over OpenRouter); instrument measurements, not listening judgements; differences under ~0.3 s are noise. **Not a v4 timing tool**: in a long-form v4 render the same tags overshot these values and drifted (4a-3). Kept as the v3 reference and as the source of the quiet-tag data. Full table, method and transcript check: `studio` -> `references/elevenlabs-tagging.md` -> "Measured pause ladder, second voice, in-passage".

| Delivered silence | v3 | v4 |
|---|---|---|
| native sentence gap (no tag) | ~1.07 s | not measured (comma ~0.3 s) |
| `[short pause]` | not measured | ~1.1 s |
| `[pause]` | not measured | ~1.0 s |
| `[long pause]` x1 | ~2.3 s | ~1.1 s |
| `[long pause]` x2 | ~3.0 s | ~1.6 s (1.3-2.4 s across identical takes; long-form median ~2.5 s, 4a-3) |
| `[long pause]` x3 | ~2.5 s (no gain over x2) | not measured |

- v3 -> v4 mapping: a v3 tag-timed rest has no reliable v4 tag equivalent: on v4 every designed rest longer than a sentence break becomes a split-map entry (4a-3 hard rule). In this probe v4 `[short pause]` = `[pause]` = `[long pause]` x1 (~1.1 s). Never stack more than 2 `[long pause]` on v3; never double them on v4.
- v4: the three singles were indistinguishable in the probe. Durations are not fixed values: x2 varied 1.3-2.4 s across identical takes (~0.5 s take-to-take spread), and lengthened across a long-form episode.
- `[quietly]`: v3 consumed with no measurable level/noise change; v4 consumed and ~9-12 dB quieter and breathy (whisper-like; voiced fraction 0.05-0.14 vs ~0.5-0.7 untagged, >4 kHz share 34-43% vs ~10%). `[whispers]`: v3 consumed, no measurable change; v4 same level as `[quietly]` (no extra step). No tag was spoken aloud on either model. Whether the v4 take reads as a soothing whisper is for the user's listening gate.
- These numbers differ from 4a-3's single-sentence `sarah` figures (4.8 s / 1.8 s): pause length depends on voice and position, so re-measure on the production voice.

#### 4a-5. Delivery-tag scope and reset (one v4 voice, 2026-10, [measured])
A scope probe on one voice (`eleven_v4`, cases A x2 takes, B, C), instrument measurements vs an untagged control; verify on your voice. Data: `studio` -> `references/elevenlabs-tagging.md` -> "Delivery-tag scope and reset".
- **Scope**: a delivery tag lasts about ONE sentence. Text-initial `[quietly]` (A) hushed sentence 1 (-8.6 / -9.5 dB vs control, two takes); sentences 2-3 came back within +0.1..+1.1 dB of control with normal voicing. Mid-text `[quietly]` (C) hushed its own sentence only (-5.2 dB); the next sentence was normal (-0.5 dB).
- **Reset**: the effect does not survive a `[long pause]` x2 stack, in either text-initial or mid-text position.

Rules:
1. Place the tag on EACH sentence (or each paragraph's sentences) it must affect. One tag at a paragraph, section or chunk head does not set a register; a sustained register re-applies its hold tag (4a-2 cue placement).
2. Never assume a register carries across a pause stack.
3. Expect a level step where untagged text follows hushed text. [measured, one v4 voice, single take] an untagged paragraph that followed hushed text measured +1.8 dB over the preceding paragraph (per-paragraph LUFS on the rendered take).
4. Fix order for a level step: (a) measure per-paragraph loudness; (b) level-match in post (`audio-post-mix` / `audio-treatment`); (c) only if the delivery itself (not the level) is wrong, use a voiced-calm tag: `[gently]` on the offending sentence is [measured, single take] (4a-6); other voiced-calm tags are [unverified] (4a-2).
5. Scope on v3 and on other voices is not measured; audition.

#### 4a-6. `[gently]` as a voiced-calm cue (one v4 voice, 2026-10, [measured, single take])
One voice, `eleven_v4` over OpenRouter, on an intro passage whose untagged notice sentence stood above its neighbours. One take per variant, no re-rolls; instrument measurements (per-paragraph loudness, voiced fraction), plus a transcript check. Variant B is the same text with only `[gently] ` inserted before the notice sentence. Verify on your voice. Data: `studio` -> `references/elevenlabs-tagging.md` -> "`[gently]` as a voiced-calm cue".

| Notice paragraph vs preceding paragraph | Step |
|---|---|
| untagged | +1.80 dB |
| Variant B (`[gently]` on the notice) | -0.07 dB |

- **Flattens the sentence to its neighbours' level.** With `[gently]` the notice sat level with the paragraph before it (-0.07 dB) instead of +1.80 dB above it; it is no longer the loudest paragraph. It resolved the level step in that test (4a-5 rule 3).
- **Not spoken aloud.** No tag text in the transcript; no unexplained audio around the tag. It was not ignored either: the step moved and the sentence read slightly slower (6.83 s).
- **Stays voiced, not a whisper.** Voiced fraction 0.56 (against 0.05-0.14 for v4 `[quietly]`, 4a-4): the opposite of the quiet-tag behaviour, so it is the voiced-calm option 4a-2 was looking for.
- **Cost:** the whole take measured quieter (-26.44 LUFS) and the opener-to-notice step stayed at +2.16 dB; level-match in post to the project target.
- **Scope:** per 4a-5, put it on each sentence it must affect. One take, one voice, one sentence position: audition before relying on it elsewhere, and do not read -0.07 dB as a guaranteed value.

#### 4a-7. Emotion tags fit the sentence type (all tag models)
This is the one home of this rule; other skills point here. An emotion tag sets the intonation of the sentence it opens, so it must fit that sentence's type.
- **Question-type tags** (`[curious]`, `[wondering]` and the like) carry a rising, questioning lift. Use them only on real questions or wondering lines, never on a flat statement or an instruction: `[curious] Start with the light.` sounds off.
- **Statements** get a neutral tag, a warm or matter-of-fact tag, or no tag.
- **Take and chunk seams:** when a new take or chunk opens on a statement, give it no tag or the same tone as the previous take's closing tag, so the seam does not change mood. This narrows "repeat the active emotion tag at each chunk start" (Step 6): repeat it only when it fits the opening sentence.
- **Tag QC before any paid render:** list every tag with the sentence it opens (tag, sentence type: question / wondering / statement / instruction, sentence text) and flag every question-type tag on a statement or instruction, and every chunk or take opening whose tag changes tone from the previous closing tag. Fix the flags before handing the text to `speech-render`; record the list with the direction.

#### 4b. Punctuation is direction (all models)
- **Period** = full stop, settled. **Comma** = short breath.
- **Ellipsis (…)** = hesitation, weight, trailing thought. `It's just… difficult.`
- **Em dash (—)** = sharp beat, interruption, self-correction. `I was going to — no. Forget it.`
- **Question mark** lifts intonation; use it to make a line sound uncertain even if it's grammatically a statement.
- **Exclamation** raises energy; one per peak, not everywhere.
- **Line breaks / short sentences** slow the pace and add gravity. Long run-on sentences speed it up and add breathlessness.
- **CAPS** = emphasis. 1 word per sentence at most, only the meaning-carrying word: `I told you NOT to open it.` Never caps whole sentences (sounds like shouting or gets weird).
- **Pauses on v4/v3**: `[pause]` / `[long pause]` tags and ellipses for short, expressive beats; a designed rest longer than a sentence break is a split on v4 (4a-3). **Never `<break>` on v4/v3.**
- **Pauses on v2 models**: `<break time="1.5s" />` (max 3s), no more than a few per generation or the audio destabilizes.

#### 4c. Text normalization (do this to every script)
Rewrite anything a voice might misread into how it should be SPOKEN:
- `$19.99` -> "nineteen ninety-nine" or "nineteen dollars and ninety-nine cents" (match the ad style)
- `555-123-4567` -> "five five five… one two three… four five six seven"
- `example.com/support` -> "example dot com slash support"
- `2026-10-06` -> "October sixth, twenty twenty-six"
- `14:30` -> "two thirty P M"; `100%` -> "one hundred percent"; `Dr.` -> "Doctor"; `Ctrl+Z` -> "control Z"
- Acronyms: decide letters vs word ("N A S A" vs "NASA", "S Q L" vs "sequel").
- Remove stage directions, speaker notes and parentheticals that aren't meant to be spoken; convert useful ones into audio tags only where the selected model supports them.

#### 4d. Pronunciation
Only fix likely misread names or technical words; list every fix so captions can be checked. Prefer a tested phonetic respelling. [verified] `<phoneme>` is supported only on `eleven_flash_v2`; [unverified] slash-delimited IPA control on v3/v4 and v3/v4 pronunciation-dictionary behavior are not established here. OpenRouter accepts `pronunciation_dictionary_locators`; see the shared reference for route placement.

#### 4e. Non-tag models (Multilingual v2, Flash, Turbo v2/v2.5)
Strip direction tags before rendering, or convert their intent to punctuation, sentence rhythm and supported settings, preserving spoken words. [inferred] Unsupported tags may be read aloud. Never add spoken narrative directions such as "she whispered" without approval of that wording change. Keep unexpressible direction in the notes.

#### 4f. A/B audition recipe (emit for any new voice or project, or before any model switch)
The measured data is one voice (`sarah`) and one sentence, so the user confirms on the project's own voice. Emit this as a short block with the output; it is a plan for authorized renders, not a render (each take is paid and goes through `speech-render` approval, `generations_count: 1` each).
1. **Line**: pick one representative line from the script (two to three sentences in the project's register; for sleep narration, a calm paragraph with one natural pause). Use the same plain text in every case.
2. **Matrix**: both models (`eleven_v3`, `eleven_v4`) x 2 takes each x the cases the project needs: (a) untagged control, (b) the profile's chosen delivery tag(s), (c) for sleep/quiet work on v4, the unverified alternatives from 4a-2 (calm punctuation only; any voiced-calm tag), (d) if pauses matter, the line with the project's pause marking. Keep voice, settings, route and text identical between models. Minimum useful set: 2 models x 2 takes x untagged + chosen tag = 8 renders; say the render count and estimated cost before asking to proceed.
3. **Listen** (user): for each take, note: is it voiced enough to feel like a person, or breathy/hissy; does it stay steady over the whole line; pause length that feels right; any tag text spoken; any unrequested stammer, breath or sound. Compare takes of the same case to separate randomness from the tag effect.
4. **Measure** (agent): integrated LUFS and true peak (`ffmpeg ... -af ebur128=peak=true`), voiced fraction and share of energy above 4 kHz if available, and the longest silent gap; compute each tag's change versus the untagged control on the SAME model and compare it with the take-to-take gap. Measurements support, never replace, the listening notes.
5. **Record** in the script header: model, tag profile, the case chosen, what stayed `unverified`, and that the user (not the agent) made the call. Re-normalize to the project LUFS target before comparing levels across models.

### Step 5 - Settings & voice
**Voice is at least as important as the text.** Always recommend a voice profile: gender/age range, timbre (warm, bright, gravelly, airy), natural energy, accent, and what it must be able to do (whisper, laugh, shout). If the ElevenLabs connector is available, call `creative_list_voices` and suggest 2-3 real voices that fit; never invent a voice_id.

[inferred] For IVCs meant for expressive work: record samples with a broad emotional range (neutral + dynamic). For a single-mood use case (sports commentary), keep samples consistent.

**Settings**: use only controls exposed by the selected route/model; the shared tagging reference holds the cited defaults and parameter rules.

| Setting | Guidance |
|---|---|
| Stability | [verified] Default 0.5; lower broadens emotional range, higher may sound monotonous. [inferred] Lower values for tag work are practitioner guidance; test the voice. |
| Similarity / style / speaker boost | [verified] Defaults 0.75 / 0 / on. Preserve profile settings; route exposure varies. |
| Stability modes | [unverified] Creative/Natural/Robust behavior on v3 vs v4; do not prescribe modes. |
| Speed | Omit for v4/v4 Turbo. On OpenRouter, 0.7-1.2 is documented only for Multilingual v2 and Flash under `provider.options.elevenlabs.voice_settings.speed`; otherwise verify support. Flag 2026-10-19: whether OpenRouter's `eleven-v4` route actually passes a speed field is [unverified] in project use; even if it does, speed is constant per call, so it cannot make a taper (use post, `audio-treatment`). |

### Step 6 - Long scripts & dialogue
- Respect project `audio.chunking.target_chars` / `audio.max_chunk_chars`, bounded by the selected endpoint's hard ceiling. Split at paragraphs/sentences, never inside a tag or sentence (the one exception is a marked hold when the project sets `audio.chunking.split_at: "phrase"`, "Split granularity" below); count the final text including tags. Repository targets: v3 ~2,700 ([observed], v3 only: a production runbook saw unreliable finalization near v3's 5,000 ceiling; do not carry it to v4), v4/v4 Turbo 4,000 ([inferred], untested choice, not a vendor figure). The sleep register uses its own band, `voice.sleep.chunk_chars` (sleep variant). Direct v4 Turbo limit is [unverified]. See the model lookup and shared reference rather than treating these targets as limits.
- [inferred] Repeat the active emotion tag at each chunk start when it fits the opening sentence (4a-7); do not repeat one-off non-verbals or SFX. Delivery tags are different: one covers only about one sentence (4a-5), so a chunk-head delivery tag colours only the chunk's first sentence. Place delivery tags per sentence where needed; in a sustained register every chunk opens with the hold tag as an identical opening string and the tag is re-applied inside long paragraphs (4a-2 cue placement).
- A chunk break is also the v4 way to get a rest longer than a sentence break: it is the 4a-3 hard rule, recorded in the split map below. Pass continuity context through `speech-render` where supported (direct TTS API `previous_text` / `next_text`; OpenRouter under `provider.options.elevenlabs`, ~500 characters each way). Never invent MCP/web fields.
- Generation is nondeterministic: compare authorized takes and use seed where supported. Creator Studio pins `generations_count: 1`; extra takes are separate authorized steps.
- **Multi-speaker**: assign a distinct profile per speaker. Text to Dialogue uses `inputs[]` with a `voice_id` and tagged `text` per turn, at most 2,000 characters total across all turns. Otherwise render mapped speaker chunks separately. Speaker labels are metadata, not spoken text. Keep a consistent emotional baseline, changing tags at beat shifts.

#### Split granularity (`audio.chunking.split_at`, opt-in)
This is the one home of the split-point rule; `speech-render`, `pace-and-pause` and `audio-treatment` point here. It moves narration pauses from cuts into a finished take to the places where generation already stops. [measured, one v4 voice, one 10-min episode] section-sized takes (43-79 s) with every comma and sentence pause cut in afterwards put inserts inside words, heard as dropouts; one passed an ASR check while sitting in a 30 ms stop closure (`pace-and-pause` → "Punctuation pause placement"). [inferred] A piece boundary needs no cut point to be found, so a pause there cannot land inside a word.
- **Key:** `audio.chunking.split_at`: `"section"` | `"sentence"` | `"phrase"`. Missing, `null` or `"section"` = the behaviour above, unchanged: entries only at chunk boundaries (length-driven and 4a-3 designed rests). The agent never sets it; the user opts in per project.
- **`"sentence"`:** the split map also gains an entry at every sentence end (as `pace-and-pause` Step 3 defines one; an ellipsis ends a sentence), so each sentence is its own render piece.
- **`"phrase"`:** as `"sentence"`, plus an entry at every comma, colon or dash the director marks as a **hold**. Marking a hold uses the existing pause-notes notation: a pause-notes row with rest type `phrase`, `after` = the sentence id plus the words up to and including the mark (`P3.s2 "where the river bends,"`), naming its split-map id. The script text stays verbatim: no inline marker, tag, ellipsis or line break (an inline marker would be spoken, and would break the `@ed` recovery check). An unmarked comma is not a split point; any pause it gets is the in-take fallback (`pace-and-pause` → "Punctuation pauses").
- **Each entry carries its rest type and target** (fields below); the lengths stay project numbers. `phrase`: `target_s` = `audio.pace.punct_pause_s.<comma|colon|dash>`. `sentence`: the pause note's value where a note covers that boundary, else `target_s: null` with `target_key` naming the key `pace-and-pause` sets it from (`audio.pace.sentence_rest_s`, or `rest_ramp` in ramp mode), since that skill varies sentence rests. A key the project lacks: `target_s: null`, `target_key: null`, flagged `user to set`; never invent a length.
- **Seams:** every piece opening is a seam for the tag QC (4a-7). [inferred] A `phrase` piece opens mid-sentence: give it no tag at its head, so the sentence does not restart its tone. Continuity, the extra requests and the listening test before first use are `speech-render` → "Split-point rendering".
- **Not in the sleep register.** `elevenlabs-voice-director-sleep` owns that register's chunking and does not read this key.

#### Split map (direction output contract)
Every direction job emits, beside the tagged chunks, a split map: the handoff to assembly (`audio-treatment` -> "Split-map assembly"). One entry per chunk boundary; on v4 every designed rest longer than a sentence break is a chunk boundary (4a-3 hard rule); with `audio.chunking.split_at` set, also every sentence end or marked hold ("Split granularity" above). Fields:
- **id**: boundary id, stable across re-renders (`b01`, `b02`, ...).
- **location**: minimal: `after_chunk` N, plus the last words before the boundary so it can be found in the text.
- **rest_type**: `sentence`, `paragraph` or `transition`; `phrase` only with `split_at: "phrase"` (a hold inside a sentence, with **mark**: `comma`, `colon` or `dash`). A designed `transition` keeps its `target_s` at pacing and is never snapped to the paragraph target (`pace-and-pause`).
- **target_s**: the pause the direction wants at that boundary. With split granularity only: **target_key** names the project key it came from, and `target_s: null` means assembly inserts nothing at that join and `pace-and-pause` sets the rest.
- **tag** / **delivered_est_s**: on v4 both `null` (no pause tag at a split; the edge silence is measured at assembly). v3 only: a tag may sit at the boundary, with its estimate from the v3 probe ladder (4a-4) as a typical value plus range.
- **owed_s**: assembly silence owed = target - delivered. Leave it `null` (or an estimate) at direction time; assembly computes it from the measured gap (`audio-post-mix`: `max(0, target - native quiet)`).

**Long-sentence guard** (generic, off by default): if `audio.pace.long_sentence_words` is set, flag every sentence over that word count (an ellipsis ends a sentence for this count) for splitting before render. Unset = no check.

Numbers in the example are illustrative targets, not measurements.
```json
{
  "episode": "<episode>",
  "file": "<episode>-splitmap.json",
  "model": "eleven_v4",
  "voice_id": "<from the project voice profile>",
  "tag_profile": "v4",
  "entries": [
    {"id": "b01", "after_chunk": 1, "line": "...and the snow keeps falling.", "target_s": 1.0,
     "tag": null, "delivered_est_s": null, "owed_s": null},
    {"id": "b02", "after_chunk": 2, "line": "...the last train has gone.", "target_s": 3.0,
     "tag": null, "delivered_est_s": null, "owed_s": null}
  ]
}
```
With `split_at: "phrase"`, a hold entry looks like this (illustrative; the target is the project's `punct_pause_s.comma`):
```json
{"id": "b07", "after_chunk": 7, "line": "...where the river bends,", "rest_type": "phrase", "mark": "comma",
 "target_s": 0.38, "target_key": "audio.pace.punct_pause_s.comma", "tag": null, "delivered_est_s": null, "owed_s": null}
```
Save it as `<episode>-splitmap.json` next to the render text. Where to split: more splits give exact pause control but more seams; longer chunks flow better but leave in-chunk rests to the voice's own phrasing. Pause-driven splits go at scene/paragraph boundaries (held beats), never mid-paragraph (shared reference -> "Pause rests, splits and the split map"), unless the project opts in to `audio.chunking.split_at` ("Split granularity" above).

#### Pause notes (direction output contract)
This is the one home of the pause notes; other skills point here. It carries the sleep director's designed rests (`elevenlabs-voice-director-sleep` → "Rest after: designed, target s") into every register. Pauses are placed **by hand after generation**, not by the model: a tag-made pause is not a timing tool (4a-3), and the user places pauses on the rendered take for control. So the director writes down where the read needs room and how much, and the pause pass places it.
- **One row** for every `land` and `turn` passage (from `@ed`, or the Step 2 read when `emotional-director` did not run), every section end, the hook, each reveal line and the closer, plus any comma or colon the read must hold longer than the voice will. Not every sentence: ordinary sentence rests are the project's `sentence_rest_s`.
- **Columns:** id (`n01`, `n02`, ... stable across re-renders) | after (sentence id and last words) | kind (`land`, `turn`, `section`, `hook`, `reveal`, `close`, `phrase`) | rest type (`phrase` inside a sentence; `sentence`, `paragraph`, `transition` between sentences; `breather` for a music-only break between sections) | target range in seconds | note (why it needs room; what the picture or music should do there).
- **Target ranges** come from the project: `audio.pace` (`punct_pause_s`, `sentence_rest_s`, `land_rest_s`) for rests in the voice, `filmcraft` → "Section breathers" lengths for breathers. Write a range, never a single value. A key that is already a range (`sentence_rest_s` min-max, breather lengths) is used as it stands. A single-value key (`punct_pause_s.comma`, `land_rest_s`) is the floor: the range runs from it to the next tier's floor (a comma up to a colon or dash, a `land` rest up to the shortest breather), or to the value plus the project's `rest_gate_pct` when there is no next tier. Never invent a key the project lacks: write `user to set`.
- **Never tag text.** No row becomes a `[pause]`, `[long pause]`, ellipsis or line break added for timing. A row that is also a split point names its split-map id (`b07`). With `audio.chunking.split_at: "phrase"`, a `phrase` row is the hold mark that makes its comma, colon or dash a split point ("Split granularity").
- **Consumers:** `pace-and-pause` (its starting list for the hand-placed pass; the user adjusts any pause by ear), `visual-director` (beats and breathers in the shot plan) and `filmcraft` → "Section breathers", so a beat lands in the picture as well as the sound.

Save it as `<episode>-pause-notes.md` next to the render text.

### Step 7 - Output format (always use this)

**0. Model record (header of the render text file, always)** - `model: <id from the project voice profile>` · `tag_profile: v3 | v4 | none (non-tag model)` · `profile_basis: <date of the measured reference, one voice/one sentence>` · `unverified_tags: <list of tags/wording used that are not measured on this model>` · `timing_doctrine: <project's, tuned on model X>` and `reaudition_needed: yes/no` · `ab_audition: done by user on <voice> | not done`. A script without this header is not ready for `speech-render`.

**1. Read summary** - one line: use case, model, tone, length estimate (~150 spoken words per minute at 1.0 speed).

**2. Emotional arc (visual)** - an energy sparkline across beats plus labels, e.g.:
```
Energy  ▂▂▃▅▃▁▆█▄▂
Beat    1 2 3 4 5 6 7 8 9 10
        calm → doubt → TURN(6) → PEAK(8) → resolve
```

**3. Beat map table** - Beat | Line starts | Emotion / subtext | Energy | Delivery choice | Tags used.

**4. The prompt** - in a single code block, ready to paste. Nothing in it that shouldn't be spoken except tags (v4/v3) or `<break>` (v2).

**5. Settings & voice** - model, voice recommendation (or real picks from the connector), stability/speed/etc.

**6. Changes made** - bullet list of normalizations, pronunciation fixes, removed stage directions.

**7. Alt takes** - for the 1-3 most important lines (hook, turn, closer), give 2 alternative tag/punctuation readings so the user can A/B.

**8. Troubleshooting tips** relevant to this script (only the ones that apply).

**9. Split map** - the `<episode>-splitmap.json` from Step 6 (always, even when it holds only plain chunk boundaries).

**10. Pause notes** - the `<episode>-pause-notes.md` table from Step 6 (always for narration with picture; optional for a short ad or IVR line).

### Step 8 - Render (via `speech-render`)
This skill does not spend credits. Run the tag QC (4a-7) first. When the user wants audio, hand the render text and voice profile to
`speech-render` through the `studio` flow: it builds the call (`creative_generate_speech`, the project's model,
`generations_count: 1`), runs the `estimate_only` preflight, and calls once the render is authorized. Never re-call to retry (it charges again): poll the status. Outputs are persisted with `asset-persist`.
Without that authorization, just deliver the prompt.

---

## Genre recipes
- **Ad / promo (15-60s)**: hook in the first 3 seconds at energy 4, drop to 3 for the value, peak at the offer, warm and clear on the CTA. Brand name and CTA get CAPS or a [short pause] before them. `[excited] Big news! Our fall sale starts TODAY. [warmly] Up to fifty percent off… but only this weekend.`
- **Narration / audiobook**: neutral baseline, tags only at real emotional shifts; let punctuation carry most of it. Character voices via consistent tags per character (`[low, gravelly voice]`). Avoid over-acting the narrator.
- **Explainer / YouTube**: conversational 3, curious lifts on questions, [short pause] before reveals, smile in the voice on jokes.
- **Drama / character**: full emotional range, subtext-driven tags (`[trying not to cry]`, `[forced cheerfulness]`), breaths and reactions, strong contrast between beats.
- **Horror / thriller**: low energy baseline, [whispers]/[cautiously], short sentences, ellipses, a single sharp peak.
- **Comedy**: [deadpan] or [sarcastic] baselines, timing via em dashes and [pause] before punchlines, [laughs] only after the joke lands, never before.
- **Meditation / sleep**: use `elevenlabs-voice-director-sleep` for a whole episode. In short: no CAPS, no exclamations, short sentences on both models. On v4 profile: one project hold tag (`voice.sleep.hold_tag`) at every paragraph and chunk start and on each paragraph's second sentence, re-applied every ~2 sentences / ~250 characters; `[whisper]` / `[softly]` only in the closing lines and only if the project lists them (`voice.sleep.endings_tags`, default none); label any `[calm]`-style tag `unverified`; no pause tags for long rests: designed rests longer than a sentence break become splits with the rest inserted in assembly (room tone or digital silence, per project; 4a-3 hard rule); level to the project LUFS target in post. On v3 profile (v3-era practice): `[softly, slowly]` at the top and at section starts, pauses per the project's timing doctrine, audition everything (v3 scope is unmeasured).
- **Agent / IVR**: friendly-neutral, minimal tags, prioritize clarity and normalization, Turbo model for latency.

---

## Pitfalls to avoid
- [inferred] Overuse or clashing tags on the same words can blur performance; start sparse.
- [inferred] Body-action descriptions ([smiles], [leans in]) are less useful than audible qualities.
- Unsupported model or malformed brackets: [inferred] tags may be read aloud. Confirm `tags_supported`; no `<break>` on v3/v4.
- Sending `speed` to v4/v4 Turbo or `instructions` to ElevenLabs via OpenRouter; put direction in tagged input.
- Treating nondeterministic tag pacing as exact silence; use post for measured pauses.
- Writing tags without first reading the target model; reusing a v3-tuned tag set or pause map on v4 (or the reverse); using `[whisper]` / `[softly]` as a sleep body tag, or directing a whole sleep episode here instead of in `elevenlabs-voice-director-sleep`.
- Assuming a delivery tag's register carries over a paragraph, chunk head or `[long pause]` stack (it lasts about one sentence, 4a-5); doubling `[long pause]` on v4 or stacking more than 2 on v3; quoting ladder durations as fixed values; writing a designed long rest as a pause tag instead of a split-map entry (4a-3).
- Reaching for a tag to fix a level step before measuring per-paragraph loudness and level-matching in post (4a-5 fix order).
- Presenting an unmeasured tag as verified, or a measurement as a listening judgement.
- ALL CAPS sentences; multiple exclamation marks.
- A whole read at one energy level - always build contrast.
- Contradicting the text (a [laughs] on a grief line) unless the subtext genuinely calls for it; a question-type tag on a statement (4a-7).
- Changing the user's words. You may add tags, punctuation, CAPS and normalization; if you think a word change would help delivery, suggest it separately, never silently.
- Using voices of real people without consent, or writing prompts meant to impersonate someone.

---

## Worked example
**Input script** (30s spot, single voice):
> Every morning I told myself I'd start tomorrow. Then tomorrow became next year. Last spring I finally walked into Riverside Gym. Now I run 5K before breakfast. Your tomorrow starts today. Riverside Gym. Visit riversidegym.com

**Arc**
```
Energy  ▂▁▃▅▆▄
Beat    1 2 3 4 5 6
        regret → low → TURN(3) → pride(4) → PEAK(5) → warm CTA
```

**Prompt (profile selects eleven_v4)**
```
[tired, a little rueful] Every morning, I told myself I'd start… tomorrow. [sighs] Then tomorrow became next year.
[short pause] [quietly hopeful] Last spring, I finally walked into Riverside Gym.
[proud, energy rising] Now? I run five K… before BREAKFAST. [chuckles]
[confident and warm] Your tomorrow starts TODAY.
[short pause] Riverside Gym. Visit riverside gym dot com.
```
**Changes**: "5K" -> "five K"; URL spoken form; added ellipses for the regret beat.
**Settings** (example assumes the profile selects v4): eleven_v4, stability 0.5 where exposed, omit speed; voice: 30s-40s, warm, grounded, can sigh and smile convincingly.
---

## In Creator Studio
- **Takes:** the selected script (with the `emotional-director` layer when that stage ran), the project's voice profile or cast, and the model it pins.
- **Makes:** the voice profile (label, provider, voice_id, model, settings, source), a cast (one voice per speaker
  label, every label covered) for dialogue, and the tagged render text for the profile's model (tags only from the
  project's approved list; timing per the project's timing rule), shown with the Step 7 read summary, arc, beat map,
  changes and alt takes, plus the split map that assembly consumes and the pause notes the pause pass, the picture
  and the breathers consume (Step 6).
- **Tool:** Claude (free); voices from `creative_list_voices` (read-only). Run `ai-voiceover` first when the script
  reads like eye-copy, and its voice-ethics gate before any render. Rendering is `speech-render` (paid, per-chunk
  estimate and `estimate_only` first; test-first where the project says so).
- **Save:** put the profile and render text in the production folder and add a line to `status.md`.

## Notes
- voice_id only from `creative_list_voices` (read-only) or the project file; never from memory.
- Open items (all [unverified] / [inferred]; do not present as rules): `[calm]` / `[slowly]` as voiced-calm tags (may shift voicing, be ignored, or be spoken; `[gently]` is [measured, single take], 4a-6); whether OpenRouter's `eleven-v4` route passes a speed field (constant per call even if present); `max_chunk_chars: 4000` for v4 is [inferred]; whether v3 under-delivery still justifies doubled pause tags on v3.
- Name pronunciations and respellings are project decisions: record them in the project, never here.
- A project's own measurements, decisions and evidence belong in that project's docs, not in this skill. Every `[measured]` number here is from one or two voices, mostly single takes: verify on your voice and model.
- Source: the author's original skill `elevenlabs-voice-director` (a single SKILL.md, no references
  or scripts), copied 2026-10-08 and adapted to Creator Studio: method, model table, tag library, settings, genre
  section, pitfalls and worked example kept; model choice bound to the project voice profile; step 8 rendering moved
  to `speech-render` behind approval. It held no project-specific constants. 2026-10-20: project-specific
  evidence IDs and values removed (kept in that project's own docs); v4 pause doctrine updated to split points.
