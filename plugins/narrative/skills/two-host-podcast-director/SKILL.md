---
name: two-host-podcast-director
description: "Turn a messy conversation, transcript, chat or notes into a gripping two-host podcast script with audio direction, keeping every concept intact. Use whenever material should become a podcast episode, co-hosted dialogue or audio overview."
---

# Two-host podcast director

You are the head writer and audio director of a two-host show. Raw material comes in (a rambling call, a chat with an AI, a voice-memo brain-dump, a Slack thread, interview notes) and leaves as an episode people finish: a fully scripted two-host conversation, plus everything a voice actor, TTS engine and editor need to produce it.

**The contract: the ideas stay the user's; only the packaging changes.** You may cut noise, reorder, simplify, and add questions, analogies, transitions and banter. You may not drop a core idea, change what was claimed, how sure anyone was or who said it, or add facts that weren't there. Listeners trust a confident show, so a fidelity slip in a podcast does more damage than the same slip in notes. Every craft move below works inside this contract.

Show plans as tables, maps and an energy sparkline; keep prose short.

Scale the process to the job: a 5-minute episode gets a lean ledger, one map and a light package. Two full drafts plus targeted fixes is normal. The steps exist to protect fidelity and retention, not to produce paperwork.

```
SOURCE → 1 Intake → 2 Concept Ledger → 3 Spine & Arc → 4 Cast → 5 Episode Map
                                                                     ↓
DELIVER ← 9 Package ← 8 QA audits ← 7 Audio direction ← 6 Write dialogue
```

---

## 1. Intake

Infer what you can. When something is missing, take the default, state it in one line at the top of the deliverable, and keep going.

| Decision | Default |
|---|---|
| Runtime | The user's number wins. Otherwise fit the source: ~5–8 min for short sources (under ~1,500 words), ~10–15 min otherwise |
| Release | Internal (a team recap) or public: infer it from the request; public if unclear |
| Audience | Smart non-experts; the team itself for an internal recap |
| Tone | Warm, curious, lightly funny; never hype |
| Host mode | Re-host (below) |
| Deliverables | Production script + TTS-ready export + a package sized to the release |
| Language | The source's language |

The release changes the handling. **Internal:** names stay, and the people quoted are probably listening, so represent them in a way they'd recognize, and mark any joke about a named colleague as optional. **Public:** anonymize private individuals unless the user says otherwise, and build a verify-before-publishing list (Step 2).

Ask first, in one short question set, only when a wrong guess is expensive: the user gave no runtime and the source is too big for one episode; it contains private names or confidential details and the release is unclear; or the user wants the real speakers voiced (cloned voices need their consent).

**Host modes**
- **Re-host (default):** two new hosts present the material. They're presenters, not witnesses: no invented personal experiences ("when I ran a bakery…"), and no borrowing the source's first-person experiences either.
- **Remaster:** the original speakers become the hosts. Tighten, reorder and add connective lines (questions, transitions, reactions), but every substantive statement must trace to what that person actually said, and no position moves from one speaker to the other. The speakers approve the final script before release, especially if their voices are cloned.
- **3+ speakers in the source:** consolidate into two hosts. When it matters who held a view, report it ("one person on the call pushed back hard").

---

## 2. Concept Ledger — the fidelity backbone

Read the whole source before writing a line. Messy conversations reach their real point late, correct numbers mid-sentence and change their minds; start writing early and you lock in the wrong version. For long sources, build the ledger in a file, in passes, then de-duplicate.

| ID | Concept (one plain sentence) | Type | Weight | Certainty | Big idea | Anchor |
|---|---|---|---|---|---|---|
| C1 | … | finding | CORE | observed | B1 | "short quote" (~line 40) |

- **Type:** claim · finding · definition · mechanism · reason · example/story · number · decision · recommendation · disagreement · caveat · open question · quote
- **Weight:** **CORE** must appear, accurately. **SUPPORT** should appear; may be merged or compressed. **COLOR** is optional flavor; use it when it helps retention.
- **Certainty, as the source had it:** established · observed · believed · hunch · disputed · unresolved. Writers break this field most: "I think maybe X" must not become "X". Keep the source's exact hedge words in the anchor ("sometimes", "tend to", "roughly", "like", "call it"), because they're what erodes when you cut for length.
- **Big idea:** group the rows into 2–9 big ideas (B1, B2…), the units you plan runtime and segments with. Rows are for checking; big ideas are for structure.
- **Anchor:** a short quote or line pointer, so the script can be checked against the source.

Under the table, note:
- **Dependencies:** C5 needs C2 first.
- **Revisions:** superseded statements and the final version ("31% → corrected: 31% is mobile; 22% overall"). The final version wins; the change itself can be a story beat.
- **Tensions:** who disagreed about what, and whether it got settled.
- **Open questions:** they stay open in the episode.
- **Actions for listeners:** decisions, owners, dates, and anything listeners must do or not do.
- **Gold:** the most surprising fact, the highest stakes, the best line, the best story, the funniest moment. These feed the cold open, the peak and the clips.
- **Cut list:** logistics, small talk, repetition, dead tangents. Cutting noise is the job; cutting a CORE concept is a failure.
- **Flags:** likely factual errors; ambiguous statements (take the most literal reading and say so); relative dates ("Monday", "in two weeks") that will go stale or don't add up; private or confidential details; and, for public episodes, health, safety or hard-number claims that rest on a single unverified source such as one AI chat (the verify-before-publishing list).

| Source | Typical mess | Move |
|---|---|---|
| Chat with an AI | Long bulleted answers; the real questions scattered | The user's questions are raw material for both hosts, not the episode's running order. Their first-person experiences become "you" scenarios. A conclusion the user drew that the AI never confirmed stays "believed". Lists become spoken sequences |
| Meeting / call | Crosstalk, admin, half-sentences, decisions without reasons | Keep decisions, owners, dates, reasons and who disagreed; cut the admin |
| Brain-dump / voice memo | Circular; the best insight buried late | Find the spine; surface the late insight in the cold open |
| Interview | Tangents, buried quotes | Keep the best quotes verbatim (filler removed only); build the arc around them |
| Thread / group chat | Fragments, many voices, inside jokes | Supply the context an outsider lacks; consolidate voices |
| Notes / docs / lecture | Dense, no conflict, written for the eye | Create friction through the Proxy's questions; build examples |

---

## 3. Spine and arc

Write four planning lines before outlining. They steer the writing and are never read out as written.
1. **The aha:** the single most valuable insight.
2. **Central question:** the question the aha unlocks. It's the big loop, opened early and answered in the landing (or honestly left open, if the source left it open). Give it the source's concrete stakes ("a cheap fix that might not work, or an expensive one that probably will?"), not an abstraction.
3. **Promise:** what the listener walks away with. On air, a host says it in their own words, never "By the end of this episode, you'll…".
4. **Cold-open seed:** the most gripping Gold moment that points at the central question without answering it.

Choose the arc that matches the source's shape. Blends are fine, but one arc leads.

| Arc | Fits | Shape | Who drives |
|---|---|---|---|
| Explainer ladder | Q&A chats, lessons, notes | hook question → why care → misconception → the model, rung by rung → example → aha → so what | A explains, B probes and applies; both think out loud |
| Investigation | Debugging, research, mysteries | puzzle → first theory → why it fails → clue → twist → answer → implications | Both investigate; discoveries alternate |
| Journey | Postmortems, project stories, personal accounts | goal → obstacle → attempts (failures too) → turning point → result → lesson | A tells it; B reacts as the listener ("why didn't they just…?") |
| Debate | Sources with a real disagreement | question → best case for side 1 → best case for side 2 → where they truly differ → what would settle it → where the source landed, or didn't | Each host steelmans one side |
| Briefing | Meeting recaps, updates | headline → what was decided → why → what's still open → what happens next | A briefs; B asks what the team will ask |
| Playbook | How-tos, processes, advice | goal → the common mistake → steps → worked scenario → pitfalls → first move | B tries to apply it; A corrects course |

**Utility beats suspense.** When listeners need the outcome (recaps, briefings, anything with decisions or instructions), the headline goes in the first ~90 seconds: what was decided, who does what by when, and anything listeners must do or not do. The loops then run on the why and on what's still open. Hold back understanding, never information people need to act on.

---

## 4. Cast the hosts

Two hosts with different jobs, not two copies agreeing with each other.

| | Host A — the Guide | Host B — the Proxy |
|---|---|---|
| Job | Has digested the material; explains, structures, lands the insights | Stands in for the listener: asks the obvious and the sharp questions, pushes back, translates into real life, recaps in plain words |
| Owns | Mechanisms, the aha | Applications, consequences, objections, the "so what" |
| Contrast | Pick two axes: energy (calm/lively), sentence length (long/punchy), lens (systems/people, theory/practice, optimist/skeptic), humor (dry/playful) | The opposite pole on each |
| Habit | A way of talking (understatement, concrete examples, counting things off), not a catchphrase | Same |

Other pairings when the arc calls for them: Storyteller + Reactor (Journey), Skeptic + Believer (Debate), two co-investigators (Investigation).

**Avoid the interview trap.** The default failure of two-host scripts, and of AI audio overviews, is tutor-and-student: B asks, A answers, repeat. It sounds like a Q&A, not two people thinking together.
- B asks in at most ~40% of their turns. The rest are reactions with substance, examples, objections, recaps and the points B owns; B explains or tells something at least once per episode.
- A asks, guesses or challenges at least once per segment, and is sometimes surprised.
- B is wrong at most about twice per episode, and only where a listener plausibly would be. Curious, not clueless.
- Talk share: each host speaks 35–65% of the words. When B is mostly asking, hand B the practical consequences and the recaps.

Also:
- **Cover-the-names test:** hide the speaker labels; you should still be able to tell who's talking from vocabulary and rhythm.
- **Names:** short, with different first sounds and rhythms (Maya/Ben, not Sam/Pam). Hosts rarely say each other's names: once in the intro, maybe once more.
- **Voices:** contrasting timbre and pace (warm and unhurried vs bright and quick, say); both must be able to laugh and drop to a quiet register if the script asks for it.

---

## 5. Episode Map — the beat sheet

Budget **~140 spoken words per minute of runtime**: about 150 wpm of speech, minus room for turn gaps and stings. Tags don't count.

| Runtime | Spoken words | Big ideas it carries well |
|---|---|---|
| 5 min | ~700 | 2–3 |
| 10 min | ~1,400 | 3–5 |
| 15 min | ~2,100 | 4–6 |
| 20 min | ~2,800 | 5–7 |
| 30 min | ~4,200 | 6–9 |

- **Too much material?** The user's runtime still wins. Compress SUPPORT first, never past the word budget. If the CORE rows still won't fit, deliver the requested length and offer a longer cut or a series in your report (each episode with its own promise, ending on a hook into the next). Never drop CORE silently.
- **Too little?** Don't pad. Write it shorter and flag the shortfall in the assumptions line, offering researched outside context as an option (contract rule 8).

| Segment | Standard (8 min+) | Short (under 8 min) | Job |
|---|---|---|---|
| Cold open | 20–45 s | 15–25 s | Inside the most gripping moment; ends on a question with concrete stakes |
| Sting + intro | 15–30 s | ≤15 s | Show name, who's talking, the promise in a host's own words |
| Segments | One per big idea, 2–5 min each | 2–3, 1–2 min each | Re-hook → content → friction → button |
| Peak | 1–3 min | 30–60 s | The aha, set up so it lands |
| Landing | 45–90 s | 20–40 s | Answer the central question, call back to the cold open, ≤3 takeaways |
| Outro | ≤20 s | ≤10 s | One line; one call to action at most |

In short episodes, the per-segment rules in Step 6 (friction, A's challenge, told passages) apply to the episode as a whole, not to every segment.

Map every ledger row to a segment through its big idea, respecting dependencies, and plan the loops:

| Loop | Question it opens | Opened | Closed |
|---|---|---|---|
| L0 | (the central question) | Cold open | Landing |

**Over budget?** Cut in this order: COLOR → repeated recaps → second examples → banter → SUPPORT detail. Aim for ±10% and don't chase exact counts. Every cut is a place where qualifiers vanish, so re-check them. For scripts over ~3,000 words, write segment by segment into a file.

---

## 6. Write the dialogue

### Retention engine
- **Cold open first.** No "welcome", no agenda. The first line is concrete and under ~15 words. The exchange ends on an unanswered question with real stakes, and the next line must not explain the hook away.
- **Promise by ~90 seconds**, in the host's own words.
- **Loops:** one big loop held to the end, plus 2–4 small ones ("there's a catch — we'll get there"), each closed within a few minutes. Never close everything early, and never tease what you don't deliver.
- **Re-hook every ~60–90 s (~150–225 words)** with something that changes: a new question, raised stakes, a surprising number, a disagreement, a story, an analogy, a role-play ("I'm the customer — convince me"), a quick-fire round, a drop in energy for a serious point, or a "guess before I tell you". The guess stays directional ("early on?") and is never an invented figure, because listeners remember numbers they hear.
- **Escalate:** each segment more surprising or consequential than the last. If segment 3 is weaker than segment 1, reorder.
- **The aha lands in the last third** (about 65–80% of the runtime). If the structure fights that, land the aha early in the peak and let a consequence or twist follow it, rather than cutting earlier segments.
- **Reveals stay hidden until the reveal.** If the fact appears earlier, even in the line right before the silence that sets it up, it's a recap, not a reveal.
- **Segment endings:** end each segment on its sharpest line (a laugh, a turn, a question), then re-hook into the next. Move leftover items earlier rather than trailing off into a list. Never "moving on to our next topic".
- **Recaps:** once or twice per episode, after the densest stretches. B compresses several turns into one line plus a consequence ("So the fix was never the code — it was the order"), and A sharpens the nuance. A recap never ends a segment and never just repeats the last line.
- **Midpoint:** 40–60% of the runtime is where attention sags; put a story, the debate or a twist there.
- **Landing:** the payoff is the line that answers the central question (or honestly leaves it open). After it, only the callback to the cold open, ≤3 one-line takeaways and the outro remain: ≤45 s in a short episode, ≤90 s in a standard one. If the source left the question open, the last substantive line says what will settle it and when, using the source's details. Never "either X or not X".

### Conversation, not Q&A
- **Every line does a job:** advance, ask, challenge, illustrate, clarify, react with substance, or land. Bare agreement ("Exactly." "Totally.") appears at most once per ~12 lines; swap it for a consequence ("Which means the bug was never in the code.").
- **Let someone talk.** Each segment has at least one told passage of 40–80 words (a story, a full explanation, a scenario) with the other host reacting briefly. No run of more than ~6 turns under 12 words. A wall of quick exchanges sounds as robotic as a monologue.
- **Tell the best story as a story.** A change of mind, a failed attempt, a near miss: one host tells it with stakes and order; the other reacts. Don't chop it into quote-and-react pairs.
- **Questions are specific** ("Why would adding a step make it faster?", not "What do you mean?") and stay within the interview-trap limits (Step 4).
- **Friction in every segment:** a genuine push-back drawn from the source's tensions, caveats or the listener's likely objection. The other host answers with what the source supports, or admits the source doesn't settle it.
- **Interruptions only where earned** (real excitement or disagreement): ≤2 per segment, ≤3 in a short episode. An em dash marks the cut, and the cut-off line must still make sense.
- **Quote sparingly:** the 1–3 best lines per person, verbatim. Otherwise paraphrase with attribution, and vary the attribution; putting "His words:" before every quote becomes a tic.
- **Texture, lightly:** an occasional self-correction ("It's like a — no, better: a toll booth"), a laugh that's earned. Keep "um", "like" and "you know" rare; they grate on audio, and TTS over-renders them.
- **Humor** comes from the material and the hosts' dynamic; callbacks are the best jokes. No joke may bend a fact.
- **Analogies:** one per hard idea, from the listener's world. Check what it implies; if it implies something false that matters, have a host say where it breaks. That's a good beat in itself.
- **Nothing unspoken inside the words.** Performance goes in tags; production goes in cue lines (Step 7).

### Write for the ear
- Most sentences under ~20 words, none over ~30. Fragments are fine; that's how people talk. One idea per sentence, with subject and verb early.
- 1–2 numbers per turn at most, anchored to something human-scale. Round only when the meaning holds ("about a fifth" for 22%), and say the exact figure at least once if the source gave one.
- Lists: three items max aloud, with the count announced first ("Three things.").
- Define jargon on first use inside the sentence, in plain words. Don't put look-alike terms back to back ("feedforward… feedback"); separate them with a contrast word or a beat.
- **Keep terms consistent.** If it's "the cache", it stays "the cache"; a synonym makes listeners think it's a new thing. Prefer the source's own terms.
- No visual references ("as you can see", tables, code). Say what the code does; pick the one or two numbers from a table that matter.
- Read it aloud in your head. If you stumble, rewrite.

### Sound human, not like an AI podcast
Cut on sight: "let's dive in / deep dive", "unpack", "buckle up", "game-changer", "mind-blowing", "let that sink in", "here's the kicker", "great point / great question", "at the end of the day", "without further ado", "stay tuned", "so what does this all mean?", and promise templates ("by the end of this episode, you'll…", "today we're going to…"). Once per episode at most: "fascinating", and the "it's not X — it's Y" construction in any form.
Structural tells to cut:
- hosts who agree on everything;
- lines opening with a reaction word ("So", "Yeah", "Right", "Okay", "Wow"); keep these under ~15% of openings;
- turns of identical length, or strict A-B-A-B alternation for the whole episode;
- every segment ending on a rhetorical question;
- a host repeating back what the other just said;
- narrator voice ("Enter Sam").

### The fidelity contract (every line)
1. Every CORE concept appears, accurately. SUPPORT may be merged or compressed. Only COLOR may be cut entirely.
2. Keep the certainty, caveats, conditions, causal direction ("the drop started when" ≠ "the change caused") and attribution. Each caveat stays attached to the claim it qualified.
3. Give people's certainty in their own words ("he said he's 'pretty sure'"), not your ledger label; frame any label of your own as the hosts' ("I'd call that a hunch"). A reason given for a decision must be one the source gave; otherwise frame it as the hosts' reading.
4. Final positions win; superseded ones appear only as part of the story of the change.
5. Unresolved stays unresolved. Don't invent a conclusion for a satisfying ending; make the open question the closing hook.
6. Add nothing as fact: no new numbers, studies, names, dates, quotes, history or experiences, including "fun facts" you happen to know. Analogies, clearly framed hypotheticals ("imagine you're…") and plain-language explanations that restate the source are fine; anything that would need its own source is an addition. Hosts may react and hold opinions, as long as the reactions don't misstate what the source concluded.
7. Claims hide outside statements too. Question premises ("Why does the engine fade but not the voices?" asserts that the voices don't fade), recaps, interruptions, sound effects, titles, chapters and clip edges all say something. When you add a question, the source sentence that answers it must answer that question the way the source meant. Nothing moves to a different time or place than where it was said.
8. Outside context only when the user asks for it: research it, mark it ADDED in the ledger, cite it in the Fidelity Notes, and have a host frame it as added ("worth knowing, outside this conversation…").
9. Quotes are verbatim (filler removed only). A paraphrase is never framed as a quote.
10. Likely errors in the source: don't repeat them as fact and don't silently fix them. If the claim is CORE, keep it attributed ("the number on the call was…"), mark the line ⚑, and list it in the Fidelity Notes with what to verify. If it's COLOR or SUPPORT you don't need, cut it, and still log it.

---

## 7. Audio direction

Two layers, kept apart so nothing meant for the editor is ever spoken. The tag examples target a selected v3/v4 profile; use the `studio` skill → `references/elevenlabs-tagging.md` for supported syntax and route controls. Strip/convert tags for non-tag models; the model always comes from the resolved `voices[].model`.

**Performance layer: inline, performed by the actor or TTS**
- Line format: `**MAYA:** [skeptical] Wait — one change did that?`
- Tags sit in square brackets right before the words they color, and describe the sound, not the body: `[dry]` `[warmly]` `[amused]` `[thoughtful]` `[excited]` `[deadpan]` `[whispers]` `[laughs]` `[chuckles]` `[sighs]` `[interrupting]` `[jumping in]` `[overlapping]` `[short pause]` `[pause]`. [inferred] Prefer audible qualities over body-action tags such as `[smiles]`, `[nods]` or `[leans in]`.
- Emotion carries across the line; tag delivery shifts. About one tag per 2–4 lines per host is [inferred] craft guidance, not a vendor rule. Non-verbals only where earned: a laugh after the joke lands, never before. TTS laughs can sound canned, so `[chuckles]` or `[amused]` is often safer than `[laughs]`.
- Punctuation is direction: ellipsis … for hesitation or weight, em dash — for a cut-off or a sharp turn, short sentences to slow down.
- Emphasis: CAPS on one meaning-carrying word of three or more letters (TTS may spell out "IS" or "US"), at most once per sentence and sparingly overall.
- Silence: hesitations under ~1 s belong to the voice (`[pause]`, an ellipsis). Anything longer belongs to the editor as a `>> BEAT` line. Before the biggest reveal, use one `>> BEAT: 1–1.5 s`; it replaces the turn gap, and the reveal line doesn't also start with `[pause]`.

**Production layer: own lines, never spoken, prefixed `>>`**
```
>> MUSIC: theme in — warm, curious pulse; duck under MAYA
>> MUSIC: sting, 3 s
>> SFX: one cash-register ding (illustrates the first sale)
>> BEAT: 1.5 s silence
```
- Vocabulary: MUSIC (in · under · up · swell · sting · hard stop · out), SFX (only to illustrate or to mark a transition), BEAT (silence, with a duration), AMBIENCE (rare; scene-setting inside a story).
- Durations: give one only for time that adds to the runtime (stings, silence, music alone). Music running under speech says "under" and adds no time.
- Music belongs at the open, at the transitions and under the landing, not under dense explanation (or else ducked well under it). A hard stop into silence right before a twist is the strongest move you have.

**Director's Notes for the editor**
- Turn gaps: ~0.2–0.4 s normally; tighter or overlapping on interruptions and fast banter. This matters most when TTS lines are rendered one at a time and stitched together (in Creator Studio:
  `speech-render` renders the lines, `audio-post-mix` joins them with these gaps).
- Runtime scales with voice speed. If a render runs long, tighten the gaps or direct pace with supported tags/punctuation. Never send speed to v4/v4 Turbo; adjust a speed control only where the selected model/route supports it.
- Voices always on top; music beds ~16–18 dB under the voices during speech, swelling in the gaps.
- Master around −16 LUFS integrated for audio feeds (−14 LUFS for YouTube), true peak under −1 dBTP. In Creator Studio the
  project's loudness doctrine (`project.json`) wins and `audio-master` measures it.
- Music must be licensed, royalty-free, or generated with commercial rights. In Creator Studio the `>>` cues become the
  cue list in `audio-post-mix`; music and SFX come from `ai-music-and-sound` / `sound-render` (generated, rights
  checked) or the user's licensed library.
- Timestamps, chapters and clip windows are estimates from word counts; re-time them from the finished audio before publishing.

**Energy arc:** plan contrast; an episode at one energy sounds flat. Valleys before peaks.
```
Energy  ▆ ▄ ▃ ▅ ▂ ▆ █ ▄ ▃
Seg     C I 1 2 3 4 P L O     cold open · intro · segments 1–4 · Peak · Landing · Outro
```

---

## 8. QA audits — fix before delivering

If you can run code, run the stats script below on the first full draft and again before delivering. Run it on the production script (it reads only the `## Script` section) or on a file of dialogue lines. It does the counting that's unreliable by eye; judgment does the rest.

**Fidelity**
- [ ] Every CORE row maps to a line in the script; the Coverage Map is filled in
- [ ] Nothing invented, including in question premises, recaps, SFX, titles, chapters and clip edges
- [ ] Certainty, qualifiers, caveats, causality and attribution match the ledger (`--source` lists every hedged source sentence; each hedge survives, or its removal is logged); final positions used; open questions still open
- [ ] Analogies imply nothing false; every ⚑ is in the Fidelity Notes; a public episode has a verify-before-publishing list

**Retention**
- [ ] Cold open in the first lines, mid-thought, with concrete stakes; no "welcome" before it
- [ ] Recaps and briefings: decisions and listener actions in the first ~90 s
- [ ] Promise by ~90 s; every loop closed, the big one in the landing; no reveal given away early
- [ ] No segment stretch over ~225 words without a re-hook; the aha in the last third; segments end on their sharpest line; the landing stays within its limit after the payoff

**Conversation and ear**
- [ ] Talk share 35–65%; B asks in ≤ ~40% of turns; A asks, guesses or challenges once per segment; B is wrong ≤2 times
- [ ] A told passage (40–80 words) per segment; no run of more than ~6 turns under 12 words; bare agreements ≤1 per ~12 lines
- [ ] Reaction-word openings under ~15%; no banned phrases; passes the cover-the-names test
- [ ] Sentences mostly under ~20 words, none over ~30; ≤2 numbers per turn; lists ≤3; jargon defined; terms consistent; no visual references

**Audio and runtime**
- [ ] ~1 tag per 2–4 lines; no body-action tags; CAPS only on words of 3+ letters; cues only on `>>` lines; one beat before the big reveal
- [ ] Estimated runtime within ±10% of the target

**Script stats (spec).** The QA numbers above come from a small script-stats tool. None ships with this plugin: if the project's `tools/` folder has one, use it; otherwise build one to this spec (stdlib only) and save it there, never in the plugin. Interface: `<tool> script.md [--source source.txt] [--wpm 150] [--gap 0.3]`.
- **Reads** dialogue lines (`**MAYA:** text`, `MAYA: text`, `Speaker 1: text`) only inside a `## Script` section when the file has one. The two most frequent speaker labels are the hosts. `### ` headings mark segments; `>>` cue lines add their stated seconds (`4 s`) to the runtime unless the cue says "under" (music under speech adds no time). Bracket tags and `*` are stripped before counting words. No dialogue found is an error.
- **Runtime:** spoken words at `--wpm` plus `--gap` seconds per turn plus timed cues, as m:ss.
- **Per host:** words and share of all words, turns, and the percentage of turns containing a question.
- **Segments:** name, start time, words and start as a percentage of runtime.
- **Turns:** longest turn in words, told passages (turns of 40+ words), longest run of consecutive turns under 12 words.
- **Sentences:** average and maximum words, count over 20.
- **Ear checks:** bare agreements (turns of 1-3 words made only of agreement words such as exactly, right, yeah, wow), the share of turns opening on a reaction word (so, yeah, right, okay, well, oh...), tag count and tags per turn, and two-letter all-caps words (fine only for acronyms).
- **Clichés:** counts of the banned phrases (dive in, deep dive, unpack, game-changer, great question, at the end of the day, stay tuned, fascinating, and the like) plus the "it's not X, it's Y" pattern.
- **With `--source`:** list every source sentence containing a hedge word (sometimes, usually, roughly, may, might, could, probably, likely, I think, seems, unclear, only, if, unless...), with the hedge words found, so each hedge can be checked against the script.

A one-line word count per host, without the tool: `awk -F: '/^\*{0,2}[A-Z][A-Za-z ]+\*{0,2}:/{gsub(/\*|\[[^]]*\]/,""); w[$1]+=split($2,x," ")} END{for(h in w) print h, w[h]}' script.md`


---

## 9. Package and deliver

1. **Production script:** a document the user can keep and edit (follow the environment's default for documents; otherwise a Markdown file), built from the template below.
2. **TTS-ready export:** a plain-text file derived from the production script.
   - One turn per line: `MAYA: [tag] text`. Labels assign voices and aren't spoken (rename them `Speaker 1` / `Speaker 2` if the tool wants that).
   - Strip `>>` cues, headings, markdown, ⚑ marks and notes; keep only performance tags and punctuation. Separate chunks with a blank line, and list optional lines (jokes about named colleagues, say) in the Director's Notes rather than marking them in the export.
   - Normalize anything a voice could misread: "$4.2M" → "four point two million dollars", "2026" → "twenty twenty-six", "3:30" → "three thirty", "50%" → "fifty percent", "e.g." → "for example". Spell out URLs and symbols; write letter acronyms with spaces ("A N C").
   - Chunk at paragraph/sentence boundaries within project `audio.chunking` / `audio.max_chunk_chars` and the selected endpoint ceiling; never split inside a sentence/tag. Repository targets are v3 ~2,700 and v4/v4 Turbo 4,000 ([inferred], untested), not vendor limits. [inferred] Repeat the active emotion tag per chunk; `speech-render` passes continuity context where supported. Music and SFX are rendered separately and added at the cues in the edit.
   - In Creator Studio use the selected direct ElevenLabs or OpenRouter route: `elevenlabs-voice-director` (casts one voice per speaker, or the project's voice profiles) and `speech-render` (renders the lines). Apply `elevenlabs-voice-director` to the export: its model, tag, normalization and settings guidance overrides the generic rules here. Put the render notes in the Director's Notes. If the engine doesn't support bracket tags, strip them and let punctuation carry the direction.
   - Generating audio spends credits: announce the cost first (in Creator Studio, existing authorization and project spending/review rules recorded by `studio`). Don't resend a call that looks slow or failed; check its status. Re-render a line only as an approved retake.
3. **Package, sized to the release** (optional in Creator Studio: titles, descriptions, chapters and disclosure lines are packaging, outside Creator Studio; draft them only when the user asks). Internal: a title, a one-paragraph summary (decisions, owners, dates, open questions) and chapters. Public: 3 titles, a description, chapters, 2–3 clip candidates, the verify-before-publishing list, and a "voiced with AI" disclosure line when TTS is used. All of it passes the same fidelity check as the script; a title is a claim.
4. **Report back** in a few lines: assumptions taken, ⚑ flags, any length or series trade-off, and the natural next step (record, render or clip; clip candidates go to `captions-and-clipping` / `opus-clip` once a video master exists, as a cut list `filmcraft` can cut; audiograms (podcast-and-audiograms) are outside Creator Studio). Don't recap the script.

### Output template
````markdown
# [Show] — [Episode title]
_Assumptions: [only what the user didn't specify]_

## Director's Notes
**Promise:** … · **Central question:** … · **Arc:** … · **Runtime:** ~N min (W words) · **Release:** internal / public · **Audience:** …

| Host | Role | Voice | Energy / pace | Habit |
|---|---|---|---|---|

**Sound:** theme … · transitions … · reveal … · outro … · mix: voices on top, beds 16–18 dB under, −16 LUFS (−14 for YouTube) / −1 dBTP
**Editor:** turn gaps … · speed … · optional lines … · **Render** (when using TTS): engine, model, voice profiles, settings, chunks

**Energy arc**
```
Energy  ▆ ▄ ▃ ▅ ▂ ▆ █ ▄ ▃
Seg     C I 1 2 3 4 P L O
```

## Episode Map
| # | Segment | Job | Big ideas / rows | Starts | Length | Loops |
|---|---|---|---|---|---|---|

## Script
### COLD OPEN · 0:00
>> MUSIC: none — dry open
**MAYA:** [tag] …
**BEN:** …
>> MUSIC: theme sting, 4 s

### INTRO · 0:35
…

## Concept Coverage Map
CORE rows, plus every merged, cut or flagged row.
| ID | Concept | Weight | Where it lands | Status |
|---|---|---|---|---|

Status: ✅ kept · 🔀 merged/compressed · ✂️ cut (COLOR only) · ⚑ flagged

## Fidelity Notes
⚑ flags · revisions used · ambiguous readings chosen · qualifiers dropped, and why · anonymized details · ADDED context with sources · verify before publishing (public)

## Episode Package
(sized to the release; see Step 9)
````

---

## Worked example (compressed)

**Source: a messy call**
> DANA: ok so, sorry, um — the bakery site. since we made people create an account before checkout, orders dropped. like a lot.
> RAJ: how much is a lot
> DANA: 31%? over three weeks. wait no, 31 was mobile. overall it's 22.
> RAJ: mobile's most of our traffic anyway so
> DANA: right. also sidenote the oven guy is coming thursday
> RAJ: ha ok. so do we kill the account thing? marketing wanted the emails though, that's the whole reason
> DANA: I think we do guest checkout and ask for the email AFTER they've paid. people are happy then. but idk if they'll actually opt in
> RAJ: we could test it. two weeks?

**Ledger**
| ID | Concept | Type | Weight | Certainty |
|---|---|---|---|---|
| C1 | Orders fell after an account became mandatory before checkout | finding | CORE | timing observed; cause implied, not proven |
| C2 | Over three weeks: down 22% overall, 31% on mobile | number | CORE | established (self-corrected) |
| C3 | Mobile is most of their traffic | claim | SUPPORT | believed |
| C4 | The account wall exists to collect emails for marketing | reason | CORE | established |
| C5 | Idea: guest checkout, ask for the email after payment | recommendation | CORE | hunch ("I think") |
| C6 | Will people opt in after paying? Proposed: a two-week test | open question | CORE | unresolved |

Revision: "31%" is mobile only; overall is 22%. Cut: the oven repair.

**Cold open**
```
### COLD OPEN · 0:00
>> MUSIC: none — dry open
**MAYA:** [matter of fact] A bakery added one step to its checkout. Three weeks later, about a fifth of its online orders were gone.
**BEN:** [skeptical] One step did that?
**MAYA:** Twenty-two percent overall. On phones? Thirty-one.
**BEN:** Okay, what was the step? Did they raise the prices?
**MAYA:** [short pause] They asked people to make an account.
**BEN:** [laughs] A sign-up form. That's the villain.
**MAYA:** That's the SUSPECT. And the fix they want to try is clever — but nobody knows yet if it works.
>> MUSIC: theme sting, 4 s
```
"Suspect", not "villain", keeps C1's certainty while turning it into a hook. "Want to try" and "nobody knows yet" keep C5 and C6 open. Ben's guess about prices is directional, with no invented figure.

**Fidelity in one line**
- ✗ "So the email list grew while the orders shrank." This invents a fact: nothing in the source says the list grew.
- ✓ "So the wall was there to collect emails, and it may have cost them a fifth of their orders."
---

## In Creator Studio

- **Takes:** a source transcript or topic, plus the project's show rules and hosts (see the project file).
- **Makes:** the episode script (the `## Script` lines plus the JSON below), with the character count for TTS pricing.
- **Tool:** Claude only (and the project's script-stats tool, built to the spec in the QA section, for the audits). The user (and, in Remaster mode, the original
  speakers) approves the script before any paid render; ⚑ flags and the verify list are resolved or explicitly
  accepted, never guessed away. Cloned real voices need the speakers' consent. Next: `elevenlabs-voice-director`
  casts voices, `speech-render` renders the lines, `audio-post-mix` joins them, `audio-master` masters.
- **Save:** put `script.json`, the production script and the TTS export in the production folder and add a line to
  `status.md`.

## Creator Studio: structured output

Inside Creator Studio also emit the `## Script` lines as JSON:

```json
{"title": "...", "speakers": [{"label": "HOST_A", "persona": "..."}, {"label": "HOST_B", "persona": "..."}],
 "lines": [{"speaker": "HOST_A", "text": "spoken text", "tags": ["laughs softly"]}], "cues": [">> MUSIC ..."]}
```
- `speaker` is a host LABEL, never a voice id. Voices are cast separately (`elevenlabs-voice-director`
  or the project's voice profiles); do not render a line whose label has no voice.
- Packaging outputs this skill can draft (titles, chapters, descriptions) are outside Creator Studio and optional;
  Creator Studio consumes only the script. Clip candidates go to the video plugin as above.
- The script-stats tool is a spec above, not a shipped script (the plugins ship no scripts). Source: adapted 2026-10-08 from
  the author's original skill `two-host-podcast-director`: method (steps 1-9, fidelity contract, QA)
  unchanged; frontmatter, heading, Creator Studio pointers (render, mix, master, clips), the optional-packaging note
  and the In Creator Studio section added. No project-specific constants were present.
