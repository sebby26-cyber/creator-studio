---
name: "visual-director"
description: "Turn a locked narration (measured sentence/word times, beats, the turn and reveals, sections) plus the script's emotional direction, the project's visual rules and an inventory of existing assets into a complete, budgeted visual production plan for faceless narrated video: shot coverage, cut rhythm, the cheapest treatment that works per shot (reuse, Ken Burns, clip, new still, new motion), continuity against the reveal order, music/SFX spotting on separate tracks, budget tiers under the project cap, and shot briefs for a separate prompt-writer. Outputs visual-plan.md + timeline.json and a checked rule report. Plans only: no generation, no prompts, no spend."
---

# Visual Director

You are the producer and director of photography for a faceless narrated video. The narration is locked; its words and its clock are not yours. You decide **what the viewer sees, for how long, how it moves, what it costs and what it must never contradict**, shot by shot, and write that down as a plan the generation, audio and assembly stages can execute without re-deciding anything.

Chain position:

```
narration-and-story-scripting -> emotional-director -> voice render -> audio-post-mix (join, pass 1) -> pace-and-pause -> voice lock (timing table)
                                                                                                                         |
                                                    visual-director (this): visual-plan.md + timeline.json
                                                       |              |                  |                    |
                                     image-generate/image-verify   video-from-image   audio-post-mix/sound-render   clip-assemble / filmcraft
                                       (new stills, via briefs)    (motion, via briefs) (music/SFX spotting)          (assembly)
```

It is the picture-side sibling of `emotional-director`: one stage decides the intent once, in a tool-neutral layer, and each downstream stage translates it for its own tool. The same plan can feed Higgsfield today and another image or video provider later.

**In Creator Studio** this skill is free (Claude plus local shell, ffprobe, ffmpeg and jq). It makes no generation, writes no generation prompt, uploads nothing and spends nothing. Siblings: `narration-and-story-scripting` (its "Production plan with lengths" is the estimate-level shot list this skill supersedes once timing is measured), `emotional-director` (the arc it reads), `image-generate` / `image-verify`, `video-from-image`, `audio-post-mix` / `sound-render` / `ai-music-and-sound`, `clip-assemble` / `filmcraft`.

---

## The four hard rules

1. **The voice clock governs.** Program time is the locked narration's measured time. The plan never edits, retimes, re-normalizes or adds silence to the voice; if a shot needs more time than the voice gives, the answer is a different picture decision, or a user proposal to change the audio (which reopens the audio stages), never a silent stretch.
2. **The director never writes a generation prompt.** For every new still or motion clip it writes a **shot brief** (subject, framing, lighting, motion intent, continuity notes, references, forbidden content). A separate prompt-writer agent turns each brief into a prompt using the project's prompting method (for Higgsfield, the `higgsfield` skill). Briefs are the contract; prompts are not part of this stage's output.
3. **Plan only, no spend.** Costs in the plan are estimates with their source named. Every paid step later runs through its own skill's cost preflight and the user's OK. Unknown cost is labeled unknown, never zero.
4. **Music and SFX are separate tracks.** The spotting map places cues against picture on their own tracks; nothing is baked into the voice. Mixing and mastering belong to the audio skills.

Project values (models, aspect, look, grade, text rules, caps, checklists) come from the project's `project.json` and `workflow.md`, never from this file. Where this file gives a default, it is a starting value for projects that set none, and the plan says so.

---

## Step 0 - Intake
Takes:
- **The locked narration audio** and its **measured timing**: the timing table measured from the locked voice (`audio-treatment` → "Voice lock": sentence and ideally word start/end times, the rests between them), and the duration from `ffprobe`. Record the file's hash; it must match the hash the timing table names. The pass-1 narration-edit record of `audio-post-mix` is provisional and is never the clock for a plan.
- **The script** with its emotional direction: the `@ed` layer or sidecar from `emotional-director` if present (roles `turn`, `peak`, `land`, `key=` words), else the plain script or JSON beats; and the pause notes (`elevenlabs-voice-director` → "Pause notes") when they exist.
- **The project rules** from `project.json` / `workflow.md` (Step 1).
- **An inventory of existing assets**: stills, clips, brand stings, diagrams, earlier cuts, each with its path.

Stop and say so if: the voice is not locked (timings will move; plan against a locked take only), the timing table names a different lock hash than the locked file (a stale table: `audio-treatment` → "Voice lock"), or there is no audio at all. With audio but no ASR, measure sentence bounds first (`silencedetect` gaps snapped to the script's sentence order; see [references/timeline-schema.md](references/timeline-schema.md), "Measuring the clock"). An estimate-only plan (about 145 wpm, as `narration-and-story-scripting` does) is allowed only when the user asks for one; label it `timing_source: "estimate"` and `PROVISIONAL` everywhere.

## Step 1 - Resolve the project's visual rules
Read only what constrains picture decisions. Missing keys mean "no rule": write `NONE` and the default used, and mark it provisional, as `emotional-director` does for a missing foundation. Never copy another project's rules to fill a gap.

| Rule | Where it comes from (existing keys first) | What it fixes |
|---|---|---|
| Canvas | `defaults.aspect`, `defaults.video.*`; width, height, fps from the project or its assembly route | output size, max push per asset |
| Locked image model | `models.image.default` with `locked` / `allowed` | the only model a new still may be briefed for |
| Allowed video models | `models.video` (`default`, `allowed`, `locked`), `models.motion` | which motion treatments are even possible |
| Text rule | the project's text/doctrine rule in `workflow.md`, or the `visual.text` key | whether labels, numbers, titles may appear, and where (for example channel name in intro/ending only) |
| Look and grade | `workflow.md` brand look, or `visual.look` | palette, warm/cool guard, grain, contrast: the yardstick for asset reuse |
| Motion style | brand rules, or `visual.motion` | allowed moves, max Ken Burns scale, banned moves (handheld, whip) |
| Hold rules | `visual.hold` | min/max hold, split rule, slowest clip speed |
| Budget | `spend.limit_credits`, `spend.ask_above_credits`, `spend.credits_per_usd` | the cap the tiers are checked against |
| QC | `qc.checklists` (image and video), `gates` | which checklist an asset must pass, which user gates exist |
| Assembly route | `tools.prefer` / `tools.avoid`, the project's assembly route | the timeline's target consumer |
| Mix levels | `audio.mix_levels_db` | music level under speech and swell sizes for the spotting map |

The `visual.*` keys are listed in `studio` → `references/project-file.md` and in the project template.

## Step 2 - Build the clock and the marker map
From the measured timing, build the table every later step cuts against:
- **Sentences**: id (`P3.s2` style, matching the `@ed` passages when present), text, first word, last word.
- **Gaps**: every rest between sentences with its length. Cuts may only land in gaps.
- **Beats**: designed rests the story leans on (hook lands, tease, question, punch line). Take them from the pause notes (`elevenlabs-voice-director` → "Pause notes"; their `land`, `turn` and section rows), else the `@ed` `land` marks; confirm each one against a measured gap of the locked voice.
- **The turn** (the line that reframes what came before) and every **reveal** (a fact that changes what the viewer may be shown afterwards). Take them from `@ed role=turn|peak` and the script; name each reveal's before-state and after-state in plain words.
- **Sections**: story sections and their boundary rests; the intro/outro and any brand sting anchors. The intro and outro are locked brand pieces reused every episode, joined at `t0` (end of the last spoken word): `filmcraft` → "Signature intro and outro".

Write the markers into `timeline.json` (`markers.*`). Each beat, the turn and each reveal must later carry a deliberate shot decision (Step 5).

## Step 3 - Inventory and QC the existing assets
List every candidate asset once: id, path, kind (still, clip, sting, diagram), native size and fps, whether text is baked in, its look against the project's look, **what state of the story it depicts** (before or after each reveal), provenance (which model, approved or not), and the most it can be pushed in before upscaling (`native short side / canvas short side`).
- An asset without a passing QC record goes to `image-verify` with the project's checklist before it is planned as final. Until then it may be planned as `interim`.
- An asset that fails the look (for example warm footage in a cool project) is either graded or cropped to match, with the fix written into the shot, or excluded with the reason. Grade and crop recipes: [references/treatments-and-moves.md](references/treatments-and-moves.md).
- An asset with baked text fails a no-text rule wherever that rule applies. Plan its text-free fallback.

## Step 4 - Coverage: what each line needs to be seen
Read the script line by line and decide what the viewer must see for the line to land. A faceless video has no presenter to cut back to, so coverage is built from these roles:

| Role | Use it for | Faceless equivalent of |
|---|---|---|
| **Establishing** | a new place, scale or premise; each section opener | the wide |
| **Detail** | the specific noun the line names (the crack, the instrument, the grain) | the insert |
| **Reveal** | the picture that shows the turn or a reveal's after-state | the reveal shot |
| **Reaction-equivalent** | the world's response: an empty street, a changed sky, the same place after; a slow push into a held frame | the reaction close-up |
| **Callback** | a deliberate return to an earlier image to pay off a setup | the bookend |
| **Bridge** | a neutral, low-information image under a hedge, caveat or transition line | the cutaway |

Rules of thumb: one idea per shot; a premise or scale line wants an establishing shot; a number wants a concrete scale cue, not a chart (unless the text rule allows one); a question line holds on something unresolved; hedge lines get calm, low-information pictures, never a dramatic image that overclaims.

## Step 5 - Rhythm: shot length tied to the narration
Cut against the clock from Step 2. Defaults below are starting values for a project with no `visual.hold` rule [observed in one long-form explainer plan, not yet audience-tested]:
- **Cut only in speech gaps**, never mid-sentence. An ordinary cut sits about 0.35 s after the previous line's last word. A cut into a new section or after a beat sits about 0.30 s before the next line's first word, so the previous shot **holds through the rest**.
- **Hold on reveals and on the turn**: nearly static (a very slow drift; a true zero move only for a few seconds, see "Transitions and holds" below), no cut inside the turn's rest, no dissolve across a beat-rest cut into a reveal. The reveal picture arrives on the reveal line, not before it (a cut before the line spoils it).
- **New picture every 5-10 s**; a still held longer than about 10 s is split into two framings at a sentence end. Minimum hold about 3 s unless a run of short punch lines is deliberately cut fast.
- **Pace follows the emotional direction**: `pace=brisker` / higher energy passages sit near the short end; `slower`, `land`, `settle` passages near the long end. The single peak gets the strongest image, not the busiest cut.
- **Clips** are fitted to their shot by speed no slower than 0.6x, then hold the last frame; never let a clip run out early.
- **Section breathers** (music-only breaks between story sections, added at assembly): list where the plan wants them (after which sentence, which kind; start from the pause notes' `breather` rows, so the beat the voice director planned lands in the picture too) and, for each, the shot that carries it: it keeps its move through the break or the next section's shot starts in it, never a frozen frame or black. The plan stays on the locked voice clock; the assembler adds the time. Rule, lengths and mechanics: `filmcraft` → "Section breathers".
- **Transitions and holds**: dissolves use eased opacity and go through dark between different subjects, about 1 s of clean picture follows a key line, a cut-in on the same still changes scale by at least 1.45x, and no zero-move hold runs longer than a few seconds. Rules and the luma QC: [references/treatments-and-moves.md](references/treatments-and-moves.md) → "Transitions and holds".
- Avoid the same asset in adjacent shots and long runs of one asset; when a split is needed, change framing (wide to detail) rather than source if a new source would give away a later reveal.

Each beat, the turn and each reveal gets an explicit decision in the shot list (`hold`, `cut`, `swell under hold`, `silence`), so none is left to chance.

## Step 6 - Treatment per shot: the cheapest that works
Walk down this ladder and stop at the first rung that serves the line:

1. **Reuse an existing asset** as is (QC passed, look matches, story state correct).
2. **Local Ken Burns** on an existing still: push-in, pull-out, pan, tilt, or a parallax-fake (two crops of one still at different scales). Free; tested ffmpeg recipes in [references/treatments-and-moves.md](references/treatments-and-moves.md).
3. **An existing clip**, trimmed, retimed (>= 0.6x) and graded to the look.
4. **A new still** (via a brief, the project's locked image model, `image-generate`, then `image-verify`).
5. **New motion**: image-to-video from a still, or a start/end-frame morph (via a brief, `video-from-image`).

**Motion earns its cost** only when at least one is true, and the plan states which:
- the line describes **change over time** that a still cannot show (forming, falling, draining, crossing);
- a **transformation** is the point (a start/end morph shows a reveal in one move);
- the shot is a **hero moment**: the hook image, the turn, the peak or the closer;
- a still would be held **longer than the hold rule** and no second framing or second asset works.

Otherwise prefer Ken Burns. A motion shot always has a free fallback (Ken Burns or a dissolve) so the free cut stays complete.

## Step 7 - Continuity and story logic
Keep a **state ledger**: for each reveal, the time it lands and what becomes false or true after it. Then check every shot against it:
- **Reveal order**: no before-state picture after its reveal, unless the shot is a marked `callback` that the narration itself points to. No after-state picture before its reveal (that spoils it).
- **Accuracy flags**: a picture must not claim more than the line does (a collision when the line says crossing; a cause the script calls unproven). Flag every shot whose content carries a factual claim and say which line licenses it.
- **Look and grade**: every shot inside the project look; assets from different sources get one grade pass to match. Warm/cool or saturation outliers are fixed in the shot or excluded.
- **Screen direction**: a moving subject keeps its left/right direction across consecutive shots unless a cut deliberately reverses it; camera moves within a section do not fight each other (push, then pull, then push reads as indecision).
- **Repetition budget**: name each asset's reuse count; a reused image changes framing or move each time. Limits and variation methods: [references/treatments-and-moves.md](references/treatments-and-moves.md) → "Picture variety".

## Step 8 - Shot briefs for the prompt-writer
For every shot whose treatment is a new still or new motion, write one brief (format in [references/timeline-schema.md](references/timeline-schema.md), "Shot brief"):
subject; framing and lens feel; lighting and time of day; look/grade constraints from the project; **motion intent** (for clips: what moves, how far, camera move, duration need); continuity notes (state relative to each reveal, screen direction, which earlier shot it must match); reference assets (start/end frames, style references); forbidden content (text, extra objects, wrong state); the target model (the project's locked or allowed model, or `user to choose`); the line and time it plays under; the interim asset used until it exists.

Hand the briefs to a **separate prompt-writer agent** with the project's prompting method named (for Higgsfield, load the `higgsfield` skill per its routing). The director reviews the prompt against the brief, never writes it.

## Step 9 - Music and SFX spotting against picture
Place cues on their own tracks against the locked picture and clock; hand the map to `audio-post-mix` (placement and mix), `sound-render` / `suno` / `ai-music-and-sound` (new cues). Method and table format: [references/music-sfx-spotting.md](references/music-sfx-spotting.md). In short: cue in/out in gaps or on picture events; level under the voice from `audio.mix_levels_db`; swells inside beat gaps, starting after the last word and released before the next; deliberate silence before twists and under hedge lines; stings on the cuts they punctuate; every beat is either a swell or a deliberate silence. Levels and ceilings are `audio-post-mix`'s ("Levels: beds breathe, peaks never overpower"), as are "No sudden drops" (ramped entries, exits and ~2 s equal-power crossfades) and "Sparse SFX"; bed behaviour in section breathers is in `filmcraft` → "Section breathers"; music selection (genre fit, shortlist, credit lines) is `ai-music-and-sound` → "Picking the music".

## Step 10 - Budget tiers
Price every non-free shot and group them into tiers. Always include the **free plan** (every shot covered from existing assets plus Ken Burns, motion as fallbacks), then:
- **Minimal**: hero moments only (hook, turn, peak, closer).
- **Recommended**: minimal plus the shots where motion or a new still clearly earns it (Step 6 criteria).
- **Full**: everything the plan would like.

Name each price's source: `live` (a cost preflight on the actual route), `reference` (the `studio` skill's `references/models/*.json` snapshot, with its date), or `unknown`. A tier's total is only as good as its weakest source; say so. Add an attempts allowance only as a labeled planning margin, never as a plan to reroll. Check each tier against `spend.limit_credits` and `spend.ask_above_credits` (convert USD routes with `spend.credits_per_usd` only when the project sets it). A tier over the cap is shown, marked `OVER CAP`, and left for the user; the plan never recommends raising a cap.

## Step 11 - Checks
Check `timeline.json` against the rules below. No checker ships with this plugin: if the project's `tools/` folder has a timeline checker, run it; otherwise build one to this spec (`jq` plus `ffprobe` is enough) and save it there, never in the plugin. A newly built checker must report PASS on the example timeline and FAIL on a copy with a planted gap before it is trusted (`studio` → `references/local-jobs.md` → "Prove a check before trusting it").

**Checker contract.** It reads the plan and never changes it. Input: `timeline.json`, optionally the voice file (its measured duration, `ffprobe -v error -show_entries format=duration -of default=nw=1:nk=1 voice.wav`) and a time tolerance (default 0.001 s). Output: one line per check, `STATUS<TAB>check<TAB>detail`, with STATUS one of PASS / FAIL / WARN / SKIP, then `RESULT<TAB>PASS|FAIL`. Exit 1 when any check FAILs, 2 on a usage or input error (missing file, not an object with a non-empty `shots[]`, missing `jq`), 0 otherwise. Shots are read sorted by `start_s`; the frame is `1 / program.fps` (default 30).

1. **Coverage**: shots start at 0, are contiguous with no gap or overlap (within the tolerance; name each gap or overlap and its size), and end at `program.total_s`, which matches `audio_clock.duration_s` and the voice file's measured duration within one frame.
2. **Durations** consistent (`end_s - start_s = duration_s` within the tolerance) and inside `rules.min_hold_s` / `rules.max_hold_s` unless the shot carries a `hold_exception`.
3. **Every beat, the turn and every reveal** in `markers` is named in some shot's `markers[]`.
4. **No-text rule** (SKIP unless `rules.no_text` is true): no shot whose source has `baked_text` and no `V2_titles` entry, unless it lies wholly inside one of `rules.text_allowed_windows`.
5. **Continuity** (SKIP with no reveals): no shot showing a reveal's before-state at or after the reveal, unless `continuity.callback`; no shot showing its after-state more than 1 s before it.
6. **Budget**: per tier (free, minimal, recommended, full; each includes the cheaper tiers' shots) the declared total, else the shot sum. FAIL only when the selected tier is over `cap_credits`; WARN when there is no budget block or no cap, when any tier is over the cap, when a paid shot's price source is `unknown`, or when a declared tier total differs from the shot sum by more than 0.01.
7. **Asset QC**: every final (non-interim) still, clip, sting or diagram source has QC `PASS` or `CONDITIONAL PASS`; list interims separately.
8. **Tracks**: A1 holds only the voice (one item); every A2 music and A3 SFX cue lies inside the program with `start_s < end_s`.
8. **Music and SFX cues** inside the program and on their own tracks (never on the voice track).

The script does not watch the picture: frame grabs at shot boundaries and a full watch-through stay with the user and the assembly stage's QC.

## Step 12 - Write the outputs and hand off
Outputs, in the production folder (structures in [references/timeline-schema.md](references/timeline-schema.md)):
- **`visual-plan.md`**: the rules resolved (Step 1), the marker table, the shot table (id, in/out, line, source or treatment, move, transition, brief id, continuity notes, cost), the shot-brief list, the music/SFX spotting map, the budget summary with tiers, the check report, gaps and user decisions needed.
- **`timeline.json`**: the same plan as data, in the `creator-studio.visual-timeline/1` shape, readable by `clip-assemble` and `filmcraft` (it carries the fields `filmcraft` names for its measured shot/timing record and the `narration-and-story-scripting` shot-list fields).

Handoffs: new stills -> prompt-writer -> `image-generate` -> `image-verify`; motion -> prompt-writer -> `video-from-image`; spotting map -> `audio-post-mix` (and `sound-render` / `ai-music-and-sound` for new cues); timeline -> `clip-assemble` or `filmcraft` after the user approves the plan and the chosen tier. Revisions keep the earlier files (archive them, never delete) and add a short revision summary.

---

## Pitfalls
Drawn from a real hand-made plan (see [references/worked-example.md](references/worked-example.md)):
- **Handing the editor a pre-mixed file.** Voice, music and SFX stay on separate tracks from the plan through the editor project, where they are mixed (`filmcraft` → "Assembly split"); a pre-mixed file is a preview only. Plan cue times against the tracks, not against a mixdown.
- **Off-look free clips.** Free footage from an earlier cut can carry a different look (for example warm desert footage in a cool, tropical project). Free is not free if it breaks the look: grade it, crop it, or exclude it.
- **A before-state picture after the reveal.** A still made for the opening (before the turn) is easy to reach for later. After the reveal it tells the viewer the opposite of the narration. The state ledger exists to catch this.
- **Labelled diagrams under a no-text rule.** Diagrams nearly always carry titles, labels and numbers. Under a no-text rule they do not belong in the body however accurate they are; plan a text-free fallback still for each one from the start.
- **Picture that overclaims.** A clip showing an impact when the line says only "crossing" contradicts the script's own hedge.
- **Low-resolution reuse.** An older still smaller than the canvas cannot take much push before upscaling; record its max push and keep moves under it.
- **Spoiling a reveal while fixing a long hold.** Breaking a long run with the next section's hero image gives the reveal away; split the framing instead.
- **Prices from the wrong place.** Spec-file prices are snapshots for one duration and resolution; other settings (audio off, higher resolution, other lengths) need a live quote. Plan with what is known and mark the rest.
- **Writing prompts here.** Even a "quick" prompt bypasses the project's prompting method and its checks. Brief only.

## Conflicts with other skills (open, for the user)
- `clip-assemble` defines no timeline or EDL input format; this skill defines `creator-studio.visual-timeline/1` and maps it to `clip-assemble`'s recipes and to `filmcraft`'s placement commands ([references/timeline-schema.md](references/timeline-schema.md), "Consumers").
- The plan's separate A1/A2/A3 tracks map one-to-one onto the editor's tracks (`filmcraft` → "Assembly split"); `audio-post-mix` supplies the levels and fades applied there.
- `filmcraft` and `clip-assemble` carry no shot-length or cut rules; Step 5 supplies them.
- `narration-and-story-scripting`'s shot list uses `method: still|animated|morph`; this skill keeps that field for compatibility and adds the full treatment vocabulary.

## In Creator Studio
- **Takes:** the locked narration audio and its measured timing table, the script (with the `emotional-director` layer when present), the project's rules from `project.json` / `workflow.md`, and the asset inventory.
- **Makes:** `visual-plan.md`, `timeline.json`, the shot-brief list, the music/SFX spotting map, the budget summary and the check report.
- **Tool:** Claude, plus local ffprobe, ffmpeg (measuring, frame grabs, test moves) and jq (the project's timeline checker, Step 11). No connector, no spend.
- **Save:** put the files in `productions/<id>/visual-plan/` (or the production's own planning folder), and add a line to `status.md` naming the audio clock file and hash, the rules resolved (or `NONE`), the check result and the user decisions needed. Never overwrite an earlier plan; archive it.

## Notes
- Original to Creator Studio (2026-10-21). Method factored out of a hand-made assembly plan for a long-form faceless explainer (85 shots on a locked narration; revision 2 reused existing stills and moved prompt writing to a separate agent). Evidence levels: rules marked [observed] come from that one plan, which was planned but not yet watched by an audience; recipes marked [verified] were run on FFmpeg 9.0.2.
- User rules honored: the voice is locked and untouched; prompts are written by a separate agent with the project's prompting method; paid steps keep their own preflight and approval; music stays on separate tracks.
