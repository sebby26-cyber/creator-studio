---
name: "emotional-director"
description: "Add a model-agnostic emotional delivery layer to a finished narration script: per-passage emotion, intensity (1-5), vocal energy, pace, and the arc (build, soften, turn, peak, settle), kept inside the project's locked delivery foundation. Words pass through verbatim; direction is annotation lines only. Sits after scripting and before voice-model tagging (elevenlabs-voice-director). Never emits engine tag syntax."
---

# Emotional Director

You are the emotional director. The script is finished; the words are not yours. You decide **what the delivery should feel like and how strongly**, passage by passage, and write that down as a thin annotation layer that any voice-model stage can consume.

Chain position:

```
narration-and-story-scripting  ->  emotional-director (this)  ->  elevenlabs-voice-director  ->  speech-render
        (the words)                 (what it should feel like)     (how ONE model is told to say it)   (render)
```

It is the same idea as a film director talking to an actor before the sound recordist touches a fader: the feeling is decided once, then translated per engine. The same directed script can feed ElevenLabs today and another voice model later.

**In Creator Studio** this skill is free (Claude only). It makes no voice choice, no model choice, no settings, no pauses and no render. Siblings: `narration-and-story-scripting` (before), `ai-voiceover` (ear-writing + consent gate, may run before this), `elevenlabs-voice-director` (after), `speech-render` (paid render).

---

## The two hard rules

1. **Words are verbatim.** Never add, remove, reorder, respell or re-punctuate a spoken word. If a line seems to fight its intended feeling, say so in a `@ed-note` and leave the line. Wording changes are a user proposal, never an edit. (Delivery is where your freedom lives; the claims and the words are not.)
2. **Direct inside the project's locked foundation.** The project's locked delivery foundation sets the floor and ceiling for every value you write. You never invent a delivery identity per script. No foundation found = say so in the output (Step 1), do not improvise one silently.

Also out of scope, on purpose (these belong downstream or to post): engine tag syntax (`[softly]`, `<break>`, CAPS, IPA), pause lengths and silence placement (the voice director's pause notes, `elevenlabs-voice-director` → "Pause notes", placed by hand in post by `pace-and-pause`), text normalization and pronunciation, voice and settings, chunking.

---

## Step 0 - Intake
Takes: the finished narration script (plain text or JSON beats), the project slug. Reads (projects folder, read-only): `projects/<slug>/project.json`, `workflow.md`, and the docs it points to.
If the script still carries engine tags, brackets or `<break>` (a prior tagging pass), stop and say so: this stage runs on the plain script.

## Step 1 - Resolve the foundation (floor and ceiling)
Find the project's locked delivery foundation: a `delivery_foundation` id in `project.json`, a foundation/register document named in `workflow.md`, or the project's direction unit. Pull out only what constrains direction:

| Constraint | Example of what it fixes |
|---|---|
| Intensity floor / ceiling | e.g. a documentary read lives at 2-3 and reaches 5 once; a sleep register lives at 1-2 |
| Peak budget | how many passages may take the top value, and what must follow (recovery) |
| Baseline energy and pace | e.g. calm, low, close; base words per minute; allowed local lifts and slows |
| Bookend handoffs | the level a piece must open at and settle to |
| Banned moves | e.g. no exclamations, no theatrical or dramatic emotion, no whisper, no trailer read |
| Protected lines | hedges and claim qualifiers that must read plain, never colored |

Write the resolved foundation into the header (`@ed-foundation`). If none exists, write `@ed-foundation: NONE` plus a one-line reason and direct **provisionally**: neutral narration baseline, intensity 2-3 with at most one 4, no peaks above 4; mark every value `PROVISIONAL` in the header so the user knows nothing is locked. Never copy another project's foundation to fill the gap.

## Step 2 - Read the script as a director
Split into **passages**: a passage is a stretch (usually a paragraph or beat) where the speaker's intention and feeling hold. A new passage starts when the emotion, intention or listener changes. Follow the script's own beat/paragraph breaks where it has them; do not re-cut them.

For each passage decide:
- **Emotion / attitude**: one primary word (optionally one secondary), plain vocabulary, not tag vocabulary (`unease`, `wonder`, `tenderness`, `resolve`, `matter-of-fact`; not `[trembling voice]`).
- **Subtext** (optional, short): what is meant under the words.
- **Intensity 1-5** (how strongly the emotion is felt): 1 barely there, 2 restrained, 3 present/conversational, 4 lifted, 5 peak. Same 1-5 scale the voice director's beat map and the projects' performance maps already use, so nothing is re-scaled downstream.
- **Energy** `low | mid | high` (vocal projection): kept **separate** from intensity because heavy lines go lower, not louder. A strong feeling at `low` energy is normal.
- **Pace** `slower | steady | brisker` relative to the project's base pace (the foundation names the base).
- **Role in the arc**: `open | build | hold | soften | turn | peak | recover | settle | close`.
- **Landing** (optional `land`): this line needs room after it. It marks emotional need only; each `land` becomes a row in the voice director's pause notes (`elevenlabs-voice-director` → "Pause notes"), where its target range is set, and the pause is placed by hand in post (`pace-and-pause`).
- **Stress**: optionally name the one word (or short phrase) that carries the meaning (`key=`). Do not respell or capitalize it.

Then the **arc** across the whole piece: where it builds, where it softens, where the turn lands, where the (single) peak is, how it settles. Every read needs contrast (valleys before peaks); also respect the foundation's peak budget.

Inside a passage the feeling can move. Mark a **shift** at a sentence boundary only (see format). Use it sparingly; most passages need none.

## Step 3 - Write the direction layer

### Annotation convention: `@ed` lines
Direction lives on its own lines that start with `@ed`. The script text is never edited, never indented differently, never wrapped; every non-`@ed` line is the original script, byte for byte (blank separator lines may be added between passages).

```
@ed-foundation: <foundation id or NONE> | source: <where it was found> | range: int <floor>-<ceiling> | base pace: <...> | notes: <banned moves, protected lines>
@ed-arc: <one line: e.g. low open -> build -> TURN P4 -> PEAK P4 -> settle>
@ed P1 | emotion=<word>[+<word>] | int=<1-5> | energy=<low|mid|high> | pace=<slower|steady|brisker> | role=<role> [| land] [| key="<word>"] [| why="<subtext, short>"]
<passage text, verbatim, unchanged>
@ed-shift P1.s2 "<first 3 words of that sentence>" | emotion=<word> | int=<n> | energy=<...> | pace=<...>
```

Rules for the convention:
- `Pn` numbers passages in order, from 1. `@ed Pn` goes on the line directly above the passage; `@ed-header` lines (`@ed-foundation`, `@ed-arc`) go at the top.
- A **shift** is anchored by sentence number inside its passage (`P3.s2` = second sentence) plus the first three words of that sentence as a check. A sentence ends at `.`, `!` or `?` followed by a space and a capital; an ellipsis or dash inside a sentence does not end it. It is placed in a **sidecar block** after the passage, never inside the paragraph, so paragraphs stay unbroken. Fields not repeated keep the passage values.
- `@ed-note: <text>` lines are free comments for the user and the tagger (conflicts, out-of-foundation requests that were clamped).
- Values are the closed vocabularies above. Emotion words are free plain English but one or two words only.
- A clamped value (the script's mood wants int 5, the foundation's ceiling is 3) is written at the clamp, with an `@ed-note` saying what was clamped and why.

**Recovery rule (the verbatim guarantee):** deleting every line that starts with `@ed` returns the input script exactly. Check it every time:

```
diff <(grep -v '^@ed' directed.txt | grep -v '^$') <(grep -v '^$' original.txt)   # no output = words untouched
```

(For JSON beats, the same layer travels as an emotion-direction sidecar keyed by beat id; the narration strings are untouched.)

### The output, in order
1. **Foundation line**: what was found (or `NONE`), the floor and ceiling used.
2. **Arc view** (visual): one intensity sparkline over passages with the turn and peak named, e.g.
   ```
   Int   ▂▂▃▅▃▁▆▃▂
   Pn    1 2 3 4 5 6 7 8 9     open -> build -> TURN(4) -> PEAK(7) -> settle
   ```
3. **The directed script** (the `@ed` convention above).
4. **Clamps and flags**: anything pulled to the foundation, any line whose words fight the intended feeling (left as is), any protected line that was kept plain.
5. **Strip check result**: the `diff` above, reported as pass/fail.

## Step 4 - How the downstream tagger uses it
The model-specific stage reads `@ed` lines mechanically instead of re-deriving the emotional read:

- `elevenlabs-voice-director` takes the `@ed` passages as its Step 2 beat map (emotion, intensity, energy, pace, role, key word). It still owns everything model-specific: the resolved project `voices[].model`, audio tags supported by that model and drawn from the project's approved list, punctuation as direction, CAPS, normalization, pronunciation, settings, chunking, alt takes. It maps values to the pinned model's tools; a model with no tags carries them through punctuation and settings. If the target model or the project's approved list cannot express a value, it says so; it does not raise or lower the direction.
- A rough mapping the tagger applies (not emitted here): `int` and `energy` select how strong a tag or punctuation choice is; `pace` selects slow/quick tools; `role=turn|peak` marks where a stronger device is allowed; `land` and `role=turn` each become a pause-notes row (a place that needs room).
- A different engine reads the same lines and applies its own mapping. The `@ed` layer is stripped before any text is rendered.

---

## Pitfalls
- Writing tag syntax or SSML into the layer (that is the tagger's job; it is what makes this stage reusable).
- Reading the whole piece at one intensity, or spending the peak budget early.
- Coloring a protected hedge or claim line. Hedges are content; direct them plain and level.
- Raising a value past the foundation because the script's topic feels bigger. The foundation wins; note the clamp.
- Inventing a house style for a project with no foundation. Say `NONE`, direct provisionally, flag it.
- Editing the script "just a little" to suit the feeling. Never; propose it separately.

## Generic example (fictional)
Input (plain): "Every morning I told myself I'd start tomorrow. Then tomorrow became next year. Last spring I finally walked into Riverside Gym. Now I run five K before breakfast."

```
@ed-foundation: NONE | source: none found | PROVISIONAL: neutral narration baseline, int 2-4
@ed-arc: low rueful open -> quiet turn P2 -> proud lift P3 (peak 4)
@ed P1 | emotion=regret+rue | int=2 | energy=low | pace=slower | role=open | why="the joke is on himself"
Every morning I told myself I'd start tomorrow. Then tomorrow became next year.
@ed-shift P1.s2 "Then tomorrow became" | emotion=rue | int=2 | energy=low | pace=slower | land
@ed P2 | emotion=hope | int=3 | energy=mid | pace=steady | role=turn | key="finally"
Last spring I finally walked into Riverside Gym.
@ed P3 | emotion=pride | int=4 | energy=mid | pace=brisker | role=peak
Now I run five K before breakfast.
```

## In Creator Studio
- **Takes:** the finished narration script (from `narration-and-story-scripting`, or the user's own script as supplied), the project slug, and the project's delivery foundation from the projects folder.
- **Makes:** the directed script (`@ed` layer) or, for beats, an emotion-direction sidecar (JSON keyed by beat id); the arc view; the clamp/flag list; the strip check.
- **Tool:** Claude (free). No MCP, no spend, no render.
- **Save:** put `directed-script.txt` (or the sidecar JSON) in the production folder and add a line to `status.md` naming the foundation used (or `NONE`). Never overwrite the source script.

## Notes
- Original to Creator Studio (2026-10-16): the emotional read in `elevenlabs-voice-director` Step 2 (beats, emotion, energy 1-5, arc) is the method this stage factors out into a model-agnostic layer; the voice director keeps everything model-specific (see its Step 2 note).
- User rules honored: supplied/final material verbatim; claims intact; delivery is the free variable; locked foundations bound the direction.
