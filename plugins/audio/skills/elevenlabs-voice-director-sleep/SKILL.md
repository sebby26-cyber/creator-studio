---
name: "elevenlabs-voice-director-sleep"
description: "Sleep / meditation / wind-down register variant of elevenlabs-voice-director. Use it INSTEAD of elevenlabs-voice-director for sleep stories, bedtime and meditation narration, and any quiet register that must hold steady for a whole episode. Directs an ElevenLabs read for a settled, level delivery: a hold tag at every paragraph and chunk start and on every paragraph's second sentence, a pre-render blur-watchlist fix, no in-text pause tags for long rests (they become assembly silence), paragraph-boundary chunking with carry-over, a fixed seed, and a pause QC gate. All values come from the project's settings (documented keys with defaults). Keeps the parent's normalization, pronunciation, output and render handoff. Directs only; rendering is speech-render (paid, approval first)."
---

Derived from elevenlabs-voice-director, 2026-10-20. Voice- and project-agnostic: every value below is a project setting with a documented default.

# ElevenLabs Voice Director: sleep register

You are a voice director for one register: sleep, meditation and wind-down narration. Given a script, you (1) read it for paragraph structure, rests and hazards, then (2) write the exact text, settings and split map ElevenLabs needs so the read sounds **settled and level from the first minute to the last**. In this register the goal is not "acted, not read": acting is what drifts.

The core idea is unchanged from the parent: in ElevenLabs the "prompt" IS the spoken text. Every bit of direction lives in the text (tags, punctuation, line breaks), plus the voice, settings, seed and chunking.

**Evidence labels.** `[measured]` = instrument measurements (loudness, pitch, voiced fraction, silence lengths, words per minute) from the plugin authors' own tests; `[inferred]` = reasoning, not measured; `[unverified]` = not tested. **Basis of every `[measured]` number here: one ElevenLabs v4 voice, single takes per segment, one long-form sleep episode plus one follow-up stitch test. Treat the numbers as a starting point and verify on your voice and model.** Rules marked **project decides** are known open trade-offs: surface them, do not resolve them yourself.

**In Creator Studio** this skill is the director, not the renderer: the model, voice id and settings come from the project's voice profile (`project.json` `voices[]`; resolve explicit production choices into the profile for their scope), register values come from `voice.sleep.*` (table below), the render text it writes goes to `speech-render`, and every render is a paid step covered by existing authorization and project spending/review rules recorded by `studio`. Siblings: `elevenlabs-voice-director` (every other register), `ai-voiceover` (script for the ear, consent + disclosure gate), `speech-render` (render), `audio-post-mix` / `audio-treatment` / `audio-master` (assembly silence, mix, master), `pace-and-pause` (post-join timing).

**What changed from the parent** (all sections below): sleep tag vocabulary with hold / endings-only / banned / untested lists; a hold tag at every paragraph and chunk start; the arc / peak / contrast machinery switched off; a chunk-splitting rule with carry-over and a fixed seed; a full Pause rules section that takes long rests out of the tag system; an A/B settings table; updated pitfalls; a generic worked example. Normalization, pronunciation, the model record header, the split map and the render handoff are kept.

---

## Project settings (`voice.sleep`)
Read these from the project file. A missing key uses the default below; say in the read summary which defaults were used. Never borrow another project's values. The keys are listed in `studio` → `references/project-file.md`.

| Key | Default | Meaning |
|---|---|---|
| `voice.sleep.hold_tag` | `[quietly]` | The ONE hold tag used at paragraph / chunk starts and re-tags. Common alternative: `[gently]`. |
| `voice.sleep.endings_tags` | `[]` (none) | Extra tags a project explicitly allows in the episode's closing lines. Default none: the closing lines use the hold tag like the body, no whispers. |
| `voice.sleep.banned_tags` | performance / arousal tags (list in 4a) | Never used in this register. A project may add more (e.g. `[calm]`). |
| `voice.sleep.untested_tags` | `["[calm]", "[peaceful]", "[hushed]", "[slowly]", "[gentle]"]` | Usable only inside an authorized A/B, labelled `unverified`. |
| `voice.sleep.retag_every_sentences` | `2` | Re-tag inside a paragraph after this many sentences without a tag (the second sentence of every paragraph is always re-tagged regardless, 4a-1 rule 5)... |
| `voice.sleep.retag_every_chars` | `250` | ...or after this many characters, whichever comes first. `null` turns re-tagging off. |
| `voice.sleep.split_min_rest_s` | `1.5` | A designed rest at or above this becomes a split point. |
| `voice.sleep.chunk_chars` | `{ "min": 800, "target_min": 1500, "target_max": 3500 }` | Chunk size band, counting tags. Also bounded by `audio.chunking.target_chars` / `audio.max_chunk_chars` and the endpoint ceiling. |
| `voice.sleep.stability` | `0.5` | Stability for the main arm. Similarity stays at its default. |
| `voice.sleep.carry_over_chars` | `500` | Characters of `previous_text` / `next_text` each way. |
| `voice.sleep.post_silence` | `"ask"` | `allowed` / `refused` / `ask`: may assembly insert silence for long rests? |
| `voice.sleep.rest_fill` | `{ "source": "room_tone", "crossfade_ms": 50 }` | What fills an inserted rest: `room_tone` (a room-tone file), `from_program` (tone built from the program's own in-chunk quiet) or `none` (digital silence). Room tone is optional; digital silence is allowed. |
| `voice.sleep.gain_match` | `"chunk_median_lufs"` | Level-match each segment before the join; `null` = off. |
| `voice.sleep.em_dash` | `"allowed"` | `allowed` / `banned` (some personas ban em dashes). |
| `voice.sleep.base_wpm` | `null` | Measured rate for length estimates. `null` = measure on a test render; until then estimate at the parent's 150 wpm and say so. |
| `voice.sleep.qc` | `{ "noise_db": -40, "min_s": 0.3, "gap_flag_ratio": 2, "class_band_pct": 20, "join_tolerance_pct": 20 }` | Pause QC gate thresholds (4b rule 5). |
| `voice.sleep.blur_watchlist` | `[]` (missing / `null` = empty) | Word sequences known to blur (slur or fuse into other words) on this project's voice. Each entry is a string (the word sequence) or `{ "sequence": "...", "added": "YYYY-MM-DD", "source": "<where it was found>", "fix": "<optional applied fix>" }`. Checked before render and extended after the transcript audit (4e-1). |

Seed: `defaults.seed` (existing key). If it is `null`, pick one integer for the episode and record it in the header.

## OpenRouter voice choice
For route `openrouter`, resolve `models.voice` (`default` / `allowed` / `locked`) and the exact voice profile under the same production/project rules. Read `studio` → `references/models/openrouter.json` and `references/openrouter.md`. Use the chosen model's live `supported_voices`, not `creative_list_voices`: account/library ids from direct ElevenLabs may not be usable, and the voice may differ. Explain the conflict rather than silently swapping a kept voice. For ElevenLabs models on OpenRouter, the tag direction below still applies; route-specific voices, settings and preflight come from those OpenRouter references. Do not use direct voice-library lookup or MCP preflight on this route. Render through `speech-render` with OpenRouter pricing approval, not MCP `estimate_only`.

**Route caveat for this register.** A minimal OpenRouter request (`{model, input, voice, response_format}`) sends no voice settings, no seed and no continuity, so every chunk renders cold. [measured, 2026-10-20] The OpenRouter route accepted `seed`, `voice_settings` (stability) and `previous_text` / `next_text` under `provider.options.elevenlabs`: every request in a 14-request test returned HTTP 200 on the first attempt. **Always send them; never rely on "route defaults".** `speed` is rejected on v4, and `previous_request_ids` is undocumented on OpenRouter. Not yet isolated: the audible effect of each field on its own, and seed reproducibility. Record exactly what was sent; if a field cannot be confirmed in the request log, record it as `not sent`. Re-check the route documentation, since provider options can change.

## Workflow

### Step 1 - Intake (ask only what you can't infer)
- **Register check**: confirm the job is sleep / meditation / wind-down. If the script needs an arc, a peak or performance (ad, drama, explainer, comedy), stop and use `elevenlabs-voice-director`.
- **Model**: always the project voice profile's `model`; no code default. Read it before choosing a single tag. The rules below were measured on v4 only; on `eleven_v3` use them as a starting draft and audition everything (v3 has no request stitching; its pause ladder differs, see parent 4a-4). Non-tag models (v2 / Flash / Turbo): strip tags, keep the pause and chunk rules.
- **Project register and rules**: read the project's persona, delivery direction, `timing_doctrine`, loudness target and `voice.sleep.*`. They outrank this skill where they are stricter. Where a project rule conflicts with a **project decides** item below, surface the conflict; do not silently pick a side.
- **Voice**: the project's locked voice (`voices[]`) if there is one; otherwise recommend a profile (Step 5) and real picks.
- **Target length**: estimate at `voice.sleep.base_wpm`. [measured, one v4 voice] an untagged v4 read ran near 200 wpm phonation rate, far above a typical sleep target, so do not reuse the parent's 150 wpm without checking.

If the user is absent or says "just do it", state assumptions at the top; do not invent a missing voice/model selection.

### Step 2 - Script read (sleep register)
**Switched off for this register**: the arc / turn / peak requirement, "every good read has contrast", "build valleys before peaks", the energy ladder above 2, non-verbal moments (`[sighs]`, `[laughs]`...), and the alt-take hook/turn/closer format. A whole read at one low energy level is the goal, not a fault.

If `emotional-director` ran, take its passages only for hazards and for intensity, and limit intensity to 1-2. Strip `@ed` lines before rendering.

Break the script into **paragraphs**, not beats. For each paragraph record:

| Field | What to decide |
|---|---|
| Para # + first words | So the user can find it |
| Energy | 1 or 2 only |
| Rest after | `sentence` (voice's own gap) or `designed` with a target in seconds (≥ `voice.sleep.split_min_rest_s` makes it a split point, Step 6) |
| Hazards | numbers, times, abbreviations, compass points, names, foreign words, stage directions, blur-watchlist hits (4e-1) |
| Ending? | whether this is the episode's closing paragraph (only tags the project lists in `voice.sleep.endings_tags`, default none, 4a) |

Paragraph starts matter more than any other position. [measured, one v4 voice] every paragraph onset, tagged or not, lifted the next 10 s (about +0.4 to +0.6 LU and +0.6 to +1.3 semitones pitch).

### Step 3 - Choose the model
The model ALWAYS comes from `project.json` `voices[].model`. Read `studio` → `references/models/elevenlabs.json` and `references/elevenlabs-tagging.md` as in the parent. Map the model to a tag profile (`eleven_v4` / `eleven_v4_turbo` → `v4`; `eleven_v3` → `v3`; others → none) and record it. Never carry a v3-tuned sleep script to v4 unchanged: regenerate from the plain script.

### Step 4 - Write the direction into the text

#### 4a. Sleep tag vocabulary (profile `v4`)
| List | Tags | Rule | Evidence |
|---|---|---|---|
| **Hold** | `voice.sleep.hold_tag` (default `[quietly]`; `[gently]` is the common alternative) | **One per project; use only it.** | [measured, one v4 voice, n=4 events] the 10 s after a hold tag ran about −1.8 LU and about −2.9 semitones pitch range vs the chunk median; rate rose ~25 wpm. Small sample. |
| **Endings only** | `voice.sleep.endings_tags` (default none) | Only tags the project explicitly lists, only in the episode's closing lines, never in the body. With the default, the closing lines carry the hold tag like the body: no `[softly]`, no `[whispers]`. | [measured, one v4 voice] `[softly]` / `[whispers]` render breathy and mostly unvoiced (a `[softly]` line ~12% voiced vs ~55% body baseline), so they are not defaults; parent 4a-1 / 4a-2. |
| **Banned** | `voice.sleep.banned_tags`; default: every performance / arousal tag (`[excited]`, `[laughs]`, `[curious]`, `[sarcastic]`, `[shouts]`, `[gasps]`, `[chuckles]`, `[sighs]`...), compound emotion tags (`[in awe, calm]`), CAPS emphasis, `!`, `?` | Not used in this register. | Register definition. |
| **Banned in body text** | `[pause]`, `[short pause]`, `[long pause]`, any doubled pause tag | See Pause rules (4b). | 4b rule 1. |
| **Untested, do not adopt yet** | `voice.sleep.untested_tags` (default `[calm]`, `[peaceful]`, `[hushed]`, `[slowly]`, `[gentle]`) | Use only inside an authorized A/B, labelled `unverified`. | [unverified] |

Notes on the hold tags:
- `[gently]`: [measured, single take, one v4 voice] not spoken aloud, stays voiced, and flattened a sentence to its neighbours' level; see the parent 4a-6 for the general finding.
- `[quietly]`: on v4 it can act as a whisper on its own sentence (parent 4a-4: well below the untagged control, voiced fraction very low). In a full episode it measured as a calmer, still-usable read, but whether `[quietly]` at every paragraph start sounds like a whisper chain over a long listen is a listening question for the project's review gate.
- **Project decides: `[calm]`.** Some projects use `[calm]` densely; others ban it in their delivery direction. It is unmeasured here, so this skill neither adopts nor bans it: it stays on the untested list unless the project's `voice.sleep.*` lists say otherwise.

#### 4a-1. Hold-tag placement
1. **Every paragraph start** gets the project's hold tag (the measured lift point, Step 2).
2. **Every chunk start** gets the hold tag too, as part of an **identical opening string**: hold tag + the chunk's first sentence, same tag in every chunk. Since chunks start at paragraph starts (Step 6), rules 1 and 2 coincide at chunk heads; never write the tag twice.
3. Never end a chunk on a tag.
4. Do not rely on a tag lasting past about one sentence or past ~10-15 s. [measured, one v4 voice] the effect faded within ~15 s, and the early minutes of the first chunk were the most-drifted span. The drift was the voice's expressive untagged default returning (wider pitch range, louder, faster than on tagged openings), not tags pulling upward. Rule 5 applies this to the second sentence of every paragraph.
5. **Re-tag inside paragraphs**: the **second sentence of every paragraph is always re-tagged** with the same hold tag, because a quiet tag lasts about one sentence (rule 4) and an untagged second sentence rebounds louder. [measured, one v4 voice, one episode] untagged second sentences lifted the first ~10 s of their chunk by about +1.0 to +2.7 LU; the risk was highest after a short opener (under ~8 s) followed by a long second sentence. After that, add the hold tag at a sentence start once `voice.sleep.retag_every_sentences` sentences (default 2) or `voice.sleep.retag_every_chars` characters (default 250) have passed since the last tag, whichever comes first. Words unchanged. [measured, n=1 segment, one v4 voice] it removed a loudness and pitch jump mid-paragraph (about +2.1 LU / +3.7 semitones untreated → about −1.9 LU / −0.2 semitones re-tagged), and the re-tagged openers stayed voiced. It does not fix a fast paragraph-opening sentence (Open items).
6. **Project decides: dense per-sentence tagging.** The parent suggests tagging nearly every sentence; web sources say a tag "carries forward across the line" and warn that over-tagging gives "chaotic" shifts. Measured scope is short (rule 4), but no densely tagged chunk has been measured. **Default here is paragraph + chunk start, the second sentence, plus the rule-5 cadence re-tag**, a middle ground with n=1 audio evidence. Every-sentence tagging is untested; a dense-tag arm may be run in the A/B (Step 5) and the project chooses after hearing it.

#### 4b. Pause rules
[measured, one v4 voice, one episode] the steady pauses were the voice's own phrasing and the assembly-inserted rests; the unsteady ones were the tag-made rests.

| Measured (raw chunks, silencedetect ≥ 0.3 s) | n | mean ± SD | max |
|---|---|---|---|
| `[long pause]` x2 + sentence end + paragraph break | 23 | 2.36 ± 0.50 s | 3.37 s |
| `[long pause]` x1 | 3 | 2.14 ± 0.74 s | 2.79 s |
| sentence end | 132 | 0.66 ± 0.12 s | 0.97 s |
| comma / colon | 235 | 0.43 ± 0.08 s | 0.71 s |
| chunk joins (assembly-inserted) | 11 | 3.71 ± 0.85 s, all on target | 4.90 s |

These are one voice's values; your voice's will differ. Measure your own before relying on them.

Rules:
1. **No in-text pause tags for any rest longer than a sentence break on v4.** No `[pause]`, `[short pause]`, `[long pause]`, and **no doubled tags**. [measured, one v4 voice] `[long pause]` x2 delivered a median ~2.5 s against a ~1.6 s ladder estimate, about half above the ladder maximum; a single `[long pause]` gave ~2.3-2.8 s against ~1.1 s; and the same x2 tag lengthened across the episode (~1.4 s early → ~2.8 s late). Some long tagged rests also split around a breath. This overrides the v3-era doubling rule: on v4 the measured problem is over-delivery. Whether the v3 under-delivery reason still holds on v3 is untested with v4 data.
2. **Long rests are assembly silence.** A rest above a sentence break with a designed target (≥ `voice.sleep.split_min_rest_s`) becomes a **split** at that paragraph boundary plus a split-map entry. Assembly inserts `owed_s = max(0, target_s − measured edge silence)` (tail of chunk N + head of chunk N+1) at the join. Fill the owed rest per `voice.sleep.rest_fill` (`room_tone`, `from_program` or `none` for digital silence; 50 ms crossfades by default). [measured] every assembly-inserted rest in the episode and in the stitch test landed on target (within 1%) and never stacked with a tag.
   **Project decides: `voice.sleep.post_silence`.** Some personas say "never splice silence in post". The measurements favour assembly silence: it was the only long rest that held its length. This skill writes the split-map method as its rule but does **not** override a persona. If `post_silence` is `ask` (default) or the persona forbids splicing, get the user's decision before the first render and record it in the header (`post_silence: allowed | refused`). If refused, the only remaining long-rest tool on v4 is the untagged paragraph break, and long rests are capped at whatever it delivers.
3. **Short rests are the voice's own.** [measured, one v4 voice] sentence and comma gaps were steady across chunks (CV ~7-10%). Use full stops, commas and line structure; use ellipses sparingly (on that voice an ellipsis ran about as long as a comma).
4. **Never stack.** At most one pause source per gap. A paragraph break that is a split point carries no tag and no ellipsis; the sentence ending before it is a plain full stop.
5. **QC gate (after render, before assembly sign-off).** Run silencedetect (`voice.sleep.qc.noise_db`, default −40 dBFS; `min_s`, default 0.3) on each raw chunk, exclude chunk-edge quiet, classify each gap against the text (sentence end, comma/colon, ellipsis, mid-phrase, join). **Flag** any gap above `gap_flag_ratio` (default 2) × its class median, and any chunk whose class mean is outside ±`class_band_pct` (default 20%) of the episode median for that class. A flagged gap is a listening item and a re-render candidate, not an automatic fix. Re-measure joins on the assembled file: each must be within ±`join_tolerance_pct` (default 20%) of `target_s`.
6. **Split check (before render).** **Every** designed rest in the pause map must be a split point with a split-map entry. When a pause tag is removed, its rest must not be left inside a segment. [measured, one case] a removed `[long pause]` left without a split was delivered at less than half its target; splitting there and inserting the rest in assembly hit the target.

#### 4c. Punctuation (sleep register)
- **Period** = settled stop; the main pacing tool. **Comma** = short breath.
- **Ellipsis** = sparingly, for weight inside a sentence; never at a split point, never stacked with a rest.
- **Line breaks / short sentences** slow the pace. Do not chop sentences the author wrote long: that changes the words' rhythm, so suggest it separately.
- **No CAPS, no `!`, no `?`** (register default; a project persona may say the same). A question in the script stays the author's words; flag it rather than repunctuating silently.
- **Em dashes**: per `voice.sleep.em_dash` (default `allowed`; some personas ban them).
- **No `<break>` on v4/v3.** On v2 models, `<break>` up to 3 s only for short rests; long rests still go to assembly.

#### 4d. Text normalization (do this to every script)
Rewrite anything a voice might misread into how it should be SPOKEN:
- `$19.99` -> "nineteen ninety-nine"; `555-123-4567` -> "five five five… one two three… four five six seven"
- `example.com/support` -> "example dot com slash support"
- `2026-10-06` -> "October sixth, twenty twenty-six"
- `14:30` -> "two thirty P M" (or the project's spoken style, e.g. "half past two"); `100%` -> "one hundred percent"; `Dr.` -> "Doctor"
- Compass points and clock times are easy to miss in story prose: write "SW" as "south-west" and "18:42" as "eighteen forty-two" (or the project's preferred form).
- Acronyms: decide letters vs word.
- Remove stage directions, speaker notes and parentheticals that aren't meant to be spoken.

#### 4e. Pronunciation
Only fix likely misread names or technical words; list every fix so captions can be checked. Prefer a tested phonetic respelling. [verified] `<phoneme>` is supported only on `eleven_flash_v2`; [unverified] slash-delimited IPA control on v3/v4 and v3/v4 pronunciation-dictionary behavior are not established here. OpenRouter accepts `pronunciation_dictionary_locators`; see the shared reference for route placement.

#### 4e-1. Blur watchlist (pre-render check, then post-render capture)
Some word sequences blur on a given voice even when the text is correct: a weak word fuses with its neighbour at a junction and is heard as a different word (for example an unstressed function word before a word starting with the same sound, a short opener right after the hold tag, or a repeated connective inside a comma-joined series). The project keeps its own list in `voice.sleep.blur_watchlist`; this skill holds only the method.
1. **Read the list.** Each entry is a string (the word sequence) or an object `{sequence, added, source, fix}`; use `sequence` for matching. Missing or `null` = empty list (say so in the read summary).
2. **Find (before render).** Search every chunk's render text for each sequence, case-insensitive, ignoring tags and normalizing whitespace and punctuation between the words.
3. **Fix automatically.** For each hit, break the junction with the smallest change that does: first a comma (or a full stop where the clause allows it) at the junction; only if punctuation cannot break it, a minimal rewording that keeps the meaning. If the entry has a `fix`, apply that same fix when it fits. Never move a split point or a tag to make a fix.
4. **Log every fix** in the output's change list (Step 7 item 5): chunk, the original words, the fixed words, and the watchlist entry it matched. A rewording is a listed word change; punctuation-only fixes are listed too.
5. **Capture new blurs (after render).** When the transcript audit (Step 8, `speech-render` → "Transcript audit") finds a slurred or fused sequence that is not on the list, append it to `voice.sleep.blur_watchlist` as an object entry (`sequence`, `added` = today's date, `source` = production / chunk / timestamp, `fix` = the fix applied, if any) through `studio`'s project-file write path (`studio` → `references/project-file.md`). Never edit `project.json` directly from this skill, never remove or rewrite existing entries, and record the append in `status.md`.

#### 4f. Non-tag models (Multilingual v2, Flash, Turbo v2/v2.5)
Strip tags before rendering, keep the chunking and pause rules, and keep unexpressible direction in the notes. [inferred] Unsupported tags may be read aloud. Never add spoken narrative directions without approval of that wording change.

### Step 5 - Settings, seed and voice
**Voice is at least as important as the text.** Recommend a voice whose untagged default is already low-arousal: a voice with an expressive untagged default is the drift this skill fights. If the ElevenLabs connector is available, call `creative_list_voices` and suggest 2-3 real voices; never invent a voice_id. On OpenRouter use the route's `supported_voices`.

**Seed.** Use `defaults.seed` (or pick one integer) for the whole episode, send it with every chunk, and record it in the header. ElevenLabs documents seed as best effort only, and without a seed the same text can render seconds longer or shorter across takes. On OpenRouter the seed is carried (route caveat); whether it reproduces a take is untested.

**Settings: A/B arms, not defaults.** v4 exposes only Stability and Similarity: no Style, no Speed. Leave Similarity at default.

| Arm | Text | Settings | Status |
|---|---|---|---|
| A, before | the project's existing tagging (as rendered) | as rendered | baseline |
| B, after | this skill's tagging (paragraph + chunk hold tags, re-tags, no pause tags, long rests as splits) | `voice.sleep.stability` (default 0.5), fixed seed | to test |
| C, optional | as B | stability 1.0 (Robust), same seed | to test |
| D, optional | as B plus the hold tag on every sentence (dense-tag arm) | as B | to test, project's call to run |

- **Project decides: stability granularity.** Whether v4 stability is continuous or snaps to 0 / 0.5 / 1.0 is unknown, and old v3 guidance says Robust is "less responsive to directional prompts". Small nudges (e.g. +0.05) assume a continuous control and are untested on v4. This skill tests only 0.5 vs 1.0 and prescribes no nudges.
- Speed: omit on v4. A slowing taper is a post step (`pace-and-pause`, or `audio-treatment` → "Rate taper and chain order").
- Level: tags change delivery, not the target. **Gain-match every segment** per `voice.sleep.gain_match` (default: to the chunk median integrated LUFS) before the join. [measured, one v4 voice] intro segments came out about 1.5-2.6 LU low and needed a few dB of gain to match. The final (outro) chunk is exempt and keeps its rendered level into the closing taper (`audio-post-mix` → "Gain-matching chunks"); an opener lift inside a chunk is corrected by the opener level dip first (`audio-treatment` → "Opener level dip"). Then meet the project loudness target (`audio.loudness`) in post.

### Step 6 - Long scripts: chunking and carry-over

#### Chunk-splitting rule
- Split **only at paragraph boundaries that carry a designed rest** (target ≥ `voice.sleep.split_min_rest_s`). Never inside a paragraph; never inside a sentence or tag.
- Target `voice.sleep.chunk_chars.target_min`-`target_max` characters per chunk (default 1,500-3,500), counting tags; no chunk under `chunk_chars.min` (default ~800). Also respect `audio.chunking.target_chars` / `audio.max_chunk_chars` and the endpoint's hard ceiling (parent Step 6: v4 working target 4,000 is [inferred]).
- **Project decides: chunk floor.** If the floor and the designed rests disagree (two designed rests close together), say so and give the choice: merge and let the second rest be an untagged paragraph break, or accept a short chunk (a stitch test used short per-rest segments without trouble). Do not add a pause tag to bridge it.
- **Identical opening string per chunk:** hold tag + first sentence, same tag every chunk.
- No chunk ends on a tag.

#### Carry-over (request continuity)
Send continuity on every chunk after the first:
- **Direct ElevenLabs API, v4:** pass `previous_request_ids` (at most 3, each at most 2 h old) from the preceding chunks' responses. Render chunks in order and capture each response's request id. v3 does not support request stitching.
- **Otherwise** (v3, or a route without stitching): `previous_text` / `next_text`, about `voice.sleep.carry_over_chars` (default 500) characters each way, taken from the neighbouring chunks' plain text.
- **OpenRouter:** send `previous_text` / `next_text` (tags stripped) under `provider.options.elevenlabs`; carriage is confirmed (route caveat). Do not send `previous_request_ids` (undocumented on this route). Record exactly what was sent and mark `carry_over: previous_text/next_text, sent (effect not isolated)`. Never write `carry_over: on` without a request log showing it.
- Never invent MCP or web fields.

#### Split map (direction output contract, kept from the parent)
Every direction job emits, beside the tagged chunks, a split map for assembly (`audio-treatment` → "Split-map assembly"). One entry per chunk boundary. In this register every long rest IS a chunk boundary, so `tag` is always `null`. Fields: `id` (stable `b01`, `b02`...), `after_chunk`, `line` (last words before the boundary), `rest_type` (`sentence`, `paragraph` or `transition`; a designed `transition` keeps its `target_s` at pacing, `pace-and-pause`), `target_s`, `tag` (null), `delivered_est_s` (null: edge silence is measured at assembly), `owed_s` (null until assembly computes `max(0, target_s − measured edge silence)`).
```json
{
  "episode": "<episode>",
  "file": "<episode>-splitmap.json",
  "model": "eleven_v4",
  "voice_id": "<from the project voice profile>",
  "tag_profile": "v4",
  "register": "sleep",
  "hold_tag": "<voice.sleep.hold_tag>",
  "seed": 4242,
  "entries": [
    {"id": "b01", "after_chunk": 1, "line": "...and the kettle settles.", "rest_type": "paragraph", "target_s": 1.6,
     "tag": null, "delivered_est_s": null, "owed_s": null}
  ]
}
```
Save it as `<episode>-splitmap.json` next to the render text. The seed and numbers above are illustrative.

**Long-sentence guard** (off by default): if `audio.pace.long_sentence_words` is set, flag every sentence over that word count (an ellipsis ends a sentence for this count) and split it before render; list the flags in the output. Unset = no check.

### Step 7 - Output format (always use this)

**0. Model record (header of the render text file, always)** - `model:` · `tag_profile:` · `register: sleep` · `hold_tag:` · `seed:` · `stability:` (or `not sent`) · `carry_over:` (`previous_request_ids` | `previous_text/next_text` | `not sent`; add `effect not isolated` on OpenRouter) · `post_silence:` (`allowed` | `refused` | `pending`) · `unverified_tags:` · `timing_doctrine:` and `reaudition_needed:` · `ab_audition:` · `settings_defaults_used:`. A script without this header is not ready for `speech-render`.

**1. Read summary** - one line: use case, model, register, length estimate at `voice.sleep.base_wpm` (or the stated fallback).

**2. Paragraph map table** - Para | Starts | Energy (1-2) | Rest after (`sentence` / designed target) | Split? | Hazards. (No arc sparkline in this register.)

**3. The prompt** - each chunk in its own code block, ready to paste, in order. Nothing in it that shouldn't be spoken except the hold tag (and, only if the project lists any, `voice.sleep.endings_tags` in the final chunk).

**4. Settings, seed & voice** - model, voice picks, the A/B arm being rendered, seed, carry-over method and its tested/untested status.

**5. Changes made** - normalizations, pronunciation fixes, blur-watchlist fixes (4e-1: chunk, original words, fixed words, matched entry), removed stage directions, removed pause tags (with where each long rest went).

**6. A/B plan** - which arms of the Step 5 table to render, the passage, render count and estimated cost (each take is a paid `speech-render` step, `generations_count: 1`).

**7. Project decisions pending** - every **project decides** item that applies and is not already settled in the project file (post-silence, dense tagging, stability granularity, `[calm]`, chunk floor).

**8. Split map** - the `<episode>-splitmap.json` (always, even when it holds only plain chunk boundaries).

### Step 8 - Render (via `speech-render`) and QC
This skill does not spend credits. When the user wants audio, hand the render text, split map and voice profile to `speech-render` through the `studio` flow: it builds the call (the project's model, `generations_count: 1`, seed, settings and carry-over as recorded), runs the preflight, and calls once the render is authorized. Never re-call to retry (it charges again): poll the status. Outputs are persisted with `asset-persist`. After render, run the 4b QC gate and the transcript audit (no tag text spoken, no dropped words), e.g. with whisper.cpp. A flagged transcript mismatch gets one automatic re-render of that chunk inside the approved budget cap, with no user verdict step before the join; a chunk still flagged after it is marked in the final report for the user's listening gate (`speech-render` → "Transcript audit"). Newly found blurs go to the watchlist (4e-1 step 5).

### Verification (no claim without numbers)
Judge a change only on the same passage (~45-60 s, including a measured drift span) rendered under arms A and B, measured the same way: phonation rate, pauses by class, LUFS / LRA, pitch median and range, per 10 s window. Success means a smaller in-passage pitch-range and loudness rise after the hold tag and after paragraph onsets, and tag-free rests within the QC tolerance of target. Pick the reference span from your own baseline: on the measured voice, the strongest drift was 30-60 s into the first chunk.

**Method for any test:** write and lock a test plan (metrics, thresholds, cost cap) before rendering; measure arm A first; one take per segment, no retries; record the actual cost per generation.

---

## Register recipe
- **Sleep story / bedtime**: energy 1-2 throughout; one hold tag per project at every paragraph and chunk start and on every paragraph's second sentence, plus cadence re-tags; no pause tags; designed rests ≥ `split_min_rest_s` as splits + assembly silence (subject to `post_silence`); no whispers, and closing lines tagged like the body unless the project lists `endings_tags`; level to the project loudness target in post.
- **Guided meditation**: as above; breath-count rests are designed rests and go to assembly, never to tags.
- **Wind-down explainer**: if it needs any contrast or emphasis, it is not this register: use `elevenlabs-voice-director` with its 4a-2 quiet guidance.

---

## Pitfalls to avoid
- Expecting one opening tag to hold a chunk. It faded within ~15 s on the measured voice; the next paragraph onset then lifts the read again.
- Writing `[long pause]`, doubled tags or any pause tag for a long rest on v4. They overshot and lengthened chunk by chunk.
- Stacking pause sources: a tag, an ellipsis and a paragraph break at the same gap. At most one source per gap.
- Splitting inside a paragraph, or leaving a chunk under `chunk_chars.min` without telling the user.
- Ending a chunk on a tag.
- Rendering cold: no seed, no settings, no carry-over. And the reverse: claiming carry-over or seed worked on OpenRouter when the request log does not show it.
- Adding `[softly]` / `[whispers]` to the closing lines (or anywhere) when the project has not listed them in `endings_tags`, or using `[quietly]` as a whole-episode whisper (parent 4a-2).
- Leaving a paragraph's second sentence untagged: it rebounds louder once the opener's tag fades.
- Using any performance tag, compound tag, CAPS, `!` or `?`.
- Adopting `[calm]`, `[peaceful]` or `[hushed]` as if measured.
- Applying the parent's arc, contrast or alt-take rules to this register.
- Nudging stability by +0.05 as if the control were continuous.
- Splicing silence into a project whose persona forbids it without the user's recorded decision (`post_silence`).
- Treating one voice's numbers as universal. Every `[measured]` number here is one v4 voice, single takes; audition on yours.
- Changing the user's words. You may add the hold tag, punctuation and normalization; suggest any word change separately.
- Using voices of real people without consent, or writing prompts meant to impersonate someone.

---

## Worked example (illustrative text, end of paragraph 3 into paragraph 4)
**Source text as a v3-era script might have it** (the paragraph 3 → 4 rest has a 1.6 s target in the pause map):
```
She rinses the cup and sets it upside down on the cloth, the way she always does, and the small sound it makes is the last sound the kitchen will make tonight. [long pause] [long pause]

The hallway is dark and cool. Her feet know the boards that creak and the ones that don't, and she steps around the loud ones without thinking. At the foot of the stairs she rests a hand on the rail, smooth from years of hands, and waits a moment for no reason at all.
```

**Directed for this register** (hold tag `[quietly]`, seed fixed per episode):
```
...She rinses the cup and sets it upside down on the cloth, the way she always does, and the small sound it makes is the last sound the kitchen will make tonight.
```
→ split `b01`: `target_s` 1.6, `tag` null, assembly owes `max(0, 1.6 − measured edge silence)`.
```
[quietly] The hallway is dark and cool. [quietly] Her feet know the boards that creak and the ones that don't, and she steps around the loud ones without thinking. At the foot of the stairs she rests a hand on the rail, smooth from years of hands, and waits a moment for no reason at all.
```
→ split `b02` at the next designed rest.

**Changes**: removed `[long pause] [long pause]` (rest moved to the split map); added the hold tag at the paragraph/chunk start and the always-on re-tag on the second sentence (rule 5; the third sentence is one sentence and under 250 characters past that tag, so no cadence re-tag yet); words unchanged. **Settings**: arm B (`voice.sleep.stability`, fixed seed); carry-over `previous_request_ids` on direct v4, else `previous_text` / `next_text` (OpenRouter: sent, effect untested). **Caveat**: as an excerpt both pieces fall under the default ~800-character floor; in a real episode the chunk runs from one designed rest to the next.

---

## Open items (not rules)
A follow-up stitch test (one v4 voice, one take per segment) did **not** pass as a whole under its own pre-registered guardrails; the rules above keep only the parts that measurably worked.
- **Paragraph-opening sentence pace:** an opening sentence ran ~13-14% faster than its neighbours although it was tagged. Tags do not fix it; `pace-and-pause` is the post-join remedy.
- **Whisper risk on very short tagged openers** (two- or three-word sentences): near-zero voiced fraction measured.
- **Final-sentence level drop:** a re-tagged last sentence dropped about −5.5 LU in one take.
- **Rate bands for intros** may be too strict: an intro outside a ±9% band still passed a listening check.
- **n=1** per segment; take variance is not separated from the effect.
- **`[gently]` vs `[quietly]` before a spoken disclaimer**: neither is a rule here.
- Whether `[quietly]` at every paragraph reads as a whisper chain over a long listen.
- The audible effect of each OpenRouter option on its own (carriage is confirmed).
- Doubled pause tags on v3: whether the v3 under-delivery reason still holds is untested with v4 data.

## In Creator Studio
- **Takes:** the selected script, the project's voice profile and the model it pins, the project's persona, timing rules and `voice.sleep.*`.
- **Makes:** the tagged render text per chunk (hold tag from the project setting), the model record header, the paragraph map, the split map, the A/B plan and the list of pending project decisions.
- **Tool:** Claude (free); voices from `creative_list_voices` (read-only) or the OpenRouter route's `supported_voices`. Rendering is `speech-render` (paid, preflight first).
- **Save:** put the render text and split map in the production folder and add a line to `status.md`. Record any project decision made during the run in the project file, not here.

## Project decisions (do not resolve in a run)
Each is a project setting or a project-file note; ask when it is unset.
| Item | Trade-off | Where it is recorded |
|---|---|---|
| Post-silence | Split-map assembly silence vs a persona rule "never splice silence in post" | `voice.sleep.post_silence` |
| Chunk floor | ~800-character floor vs per-paragraph / per-rest segments | `voice.sleep.chunk_chars.min` |
| Dense tagging | Paragraph start + re-tag (default) vs every sentence (untested) | `voice.sleep.retag_every_*`; A/B arm D |
| `[calm]` | Some projects use it densely, others ban it; unmeasured | `voice.sleep.banned_tags` / `untested_tags` |
| Stability granularity | Continuous (small nudges) vs 0 / 0.5 / 1.0 snapping | `voice.sleep.stability` |
| Doubled pause tags on v3 | v3 under-delivery reason untested with v4 data | project timing doctrine |

## Notes
- voice_id only from `creative_list_voices`, the OpenRouter route's `supported_voices`, or the project file; never from memory.
- A project's own measurements, decisions and evidence belong in that project's docs, not in this skill.
- Source: derived from `elevenlabs-voice-director` (2026-10-20). Kept: OpenRouter voice choice, model binding, normalization, pronunciation, non-tag model handling, model record header, split map, render handoff and studio-core references. Replaced: script analysis (arc off), tag sections, pause handling, settings, chunking, output items 2 and 7, genre recipes, pitfalls, worked example.
