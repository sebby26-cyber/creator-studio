# Worked example (anonymised)

This walks the method through a generic case modelled on a real hand-made plan for a long-form faceless science explainer. Names, files and topic are invented; the numbers are shaped like the real ones so the scale is honest. The small data twin of the short version is [example-timeline.json](example-timeline.json).

## The job

- **Narration:** about 10 minutes, locked, one narrator, measured to the millisecond. 24 designed beats (1.7-2.4 s rests), one **turn** (a 2.5 s hold, then the line that reframes the whole opening), section rests of about 2 s, a branded intro sting before the story and an ending sting after it.
- **Script direction:** an `@ed` layer with `role=turn` on one passage, `peak` on one, `land` on the beats.
- **Project rules:** 16:9 canvas at 30 fps; one locked image model for any new still; video models "user to choose"; a **no-text rule** in the body (channel name allowed in the intro and ending only); a cool palette with a "no warm tones" guard; slow push/pull moves only, 1.00-1.08; a per-run credit cap and a per-step ask threshold.
- **Inventory:** two earlier still sets (one current, high resolution; one older, lower resolution and of unrecorded model), a dozen labelled diagrams, a handful of clips from an earlier cut, two brand stings.

The story's premise: the opening paints a beautiful image (call it "the shining band"); the turn reveals that the band, as pictured, cannot exist, and shows what it would really look like ("the dark band"). That reveal is **R1**: before it, shining-band images are correct; after it, they are wrong unless the narration calls back to them.

## Step by step

1. **Clock and markers.** Sentence times from the narration-edit record, checked against `silencedetect`: every beat rest and the turn hold land within a few hundredths of a second. The marker table lists the 24 beats, the turn (hold in/out, reveal in/out), R1 (before: shining band; after: dark band) and 8 section rests.
2. **Inventory and QC.** Every asset gets a row: size (the older stills top out at a 1.39x push on the canvas), baked text (all diagrams: yes), look (two earlier clips are warm desert roads: off-look), state (one older still shows the shining band in a scene that only occurs after the turn: before-state, usable only before R1), claims (one clip shows an impact when the line says only "crossing": overclaims). Assets without a QC record go to `image-verify`.
3. **Coverage.** Each line gets a role. The hook gets an establishing hero image; each premise number gets a scale cue instead of a chart; the hedge lines ("nobody has measured this") get calm bridges; the turn gets a reveal; the closing line gets a callback to the opening image.
4. **Rhythm.** 85 shots: cuts only in gaps, 0.35 s after the last word; cuts before section openers 0.30 s before the first word, so shots hold through beats. A new picture every 5-10 s; two runs of one still over 14 s are split into two framings at a sentence end. One split was not broken with the next section's hero still, because that would have spoiled a later reveal.
5. **Treatment.** Free plan first: Ken Burns on the current stills, reuse of four earlier clips (the band forming, a shadow band twice, the sky emptying), stings at both ends. The older stills fill repeats and long holds, each move kept under its push limit. All 12 diagrams are replaced by text-free fallback stills (no-text rule). Three lines need a picture nothing shows: three **new stills**, each with a brief and an interim still so the free cut is complete today.
6. **Motion candidates.** 17 shots meet a motion criterion (the hook, the turn as a start/end morph from shining band to dark band, a crossing, a draining, the closer). Each keeps its free fallback. With the video model undecided, they are listed as candidates with no model, prompt or cost; once a model is chosen they become tiered.
7. **Continuity.** The ledger catches: the older shining-band still placed after R1 (excluded); the impact clip (excluded: overclaims); the warm desert clips and a warm still (excluded or cool-graded); three older stills with saturated or warm details (graded or cropped). The one shining-band image after R1 is the deliberate callback the narration points to.
8. **Briefs.** Three still briefs and, later, one per chosen motion shot: subject, framing, lighting, look, motion intent, continuity (R1 state), references, forbidden content (text, extra objects, wrong state), target model, interim. A separate prompt-writer agent turns them into prompts with the project's prompting skill.
9. **Spotting.** Music on A2 and SFX on A3, never on the voice. Openers and closers 12 LU under speech, beds 14 LU (levels from before the ceilings in `audio-post-mix`; under them, music sits at least 15 dB under the voice on every momentary window); swells of +6 to +8 dB inside beat gaps; deliberate silence before the tease and through the whole turn; hedge lines with no music; intro riser and title hit timed to the sting. Every beat is mapped to a swell or a silence.
10. **Budget.** Free plan: 0. Tiers for motion at 5, 9 and 17 clips priced from the model reference snapshot for one model and one duration; audio-off, higher resolution and other durations marked "live quote needed". Two tiers on the dearer model exceed the per-run cap and are shown `OVER CAP` for the user. New stills about 1 credit each on the locked model, live quote before each.
11. **Checks.** Coverage 0 to the voice duration with no gaps; every beat, the turn and R1 decided; no text in the body; no before-state after R1 except the callback; free tier under the cap; every final reused asset with a QC verdict (interims listed).
12. **Outputs.** `visual-plan.md`, `timeline.json` (85 shots, titles on V2, voice on A1, music A2, SFX A3, markers, transitions, rules), the brief list, the spotting map, the budget summary, and a list of user decisions (motion model and tier, reuse of the older unapproved stills, how the final mix reaches the assembler, the ending fade).

## What the example taught (the pitfalls in SKILL.md)

- The assembler (`filmcraft` `edit_sequence`) wants one mastered mix on A1 while the plan keeps music and SFX separate: the plan states that `audio-post-mix` renders the mix from the same cue times, and the stems stay in the editable project for review. (Since superseded: every layer now stays on its own track and is mixed in the editor, `filmcraft` → "Assembly split".)
- Free clips and older stills drifted off-look (warm in a cool project); "free" assets need the same look check as paid ones.
- A before-state still was placed after the reveal in the first draft; the state ledger is what caught it.
- Labelled diagrams broke the no-text rule even though they were accurate and the draft shot list "overrode" the rule; an unapproved draft never overrides a project rule.
- Prices from a reference snapshot cover one setting only; everything else needs a live quote, and the plan says which is which.
- Writing prompts inside the plan was later moved out to a separate prompt-writer agent; the plan now carries briefs only.
