---
name: opus-clip
description: "Plan an OpusClip pipeline that turns long video (podcasts, webinars, interviews, streams, a produced final) into reviewed short clips with the CLIPS framework: choose and trim the source, let AI find candidates, inspect and rank honestly, polish the top few, ship. The agent plans the pipeline, credit math and QA; the human reviews every clip and approves; approved picks are filed as a cut list that filmcraft can cut, and exports are registered in the projects folder. Use when someone wants to use OpusClip, turn a podcast/webinar/VOD into shorts, understand the Virality Score, pick a plan, pull one moment from a long recording, or automate a clipping pipeline. Integrity spine: the Virality Score is a PREDICTION for triage, never truth; 1 credit = 1 SOURCE minute; no unreviewed auto-post pipelines; no out-of-context clips. Distinct from captions-and-clipping (general craft), descript (long-form edit), capcut (short-form polish) and filmcraft (local cutting)."
---

# OpusClip

The **clip-pipeline tool skill**: choose the source, let AI find candidates, inspect and rank honestly, polish
the top few, ship. The agent plans and QA-checks; the **human reviews every clip**; approved clips are registered
in the project's folder. Publishing them afterwards is optional and user-approved (see the end of
this file). Pairs with the
`captions-and-clipping` craft skill.

## The POV: a first-pass clip finder, not a one-click viral machine
OpusClip genuinely compresses 3–5 hours of manual clipping into minutes of processing plus review: its
**ClipAnything** engine is multimodal (visuals + audio sentiment + facial expressions), so it finds moments even
in wordless content, and natural-language moment search pulls one moment from a 2-hour VOD without scrubbing. But
the top-1% operator holds two facts the marketing won't volunteer. **(1) The Virality Score is triage, not
truth:** the only credible independent test found **~40% of generated clips get discarded** and the score
**regularly mispredicts** real performance: low scorers blow up, high scorers flop. Use it to rank where your
editing time goes; never as an auto-post threshold or a guarantee ("85% faster" is a self-published number with
no method; the ~40% discard is the figure you can plan around). **(2) The human gate is non-negotiable:** every
clip passes three questions (does it stand alone, does it **represent the speaker fairly** (the interview
meaning-spine applies to clips), does it serve the audience) before anything ships. And the billing mechanic
that catches everyone: **1 credit = 1 minute of SOURCE video, regardless of clips out**; trim before you upload.

## Read these first
1. **`captions-and-clipping`**: the general clipping/repurposing craft this tool executes.
2. The source's script or structure (**`narration-and-story-scripting`** or **`two-host-podcast-director`**; podcast
   strategy, podcast-and-audiograms, is outside Creator Studio) + the project profile (`project.json`: brand
   constants and caption style = the clip template).

## The framework: CLIPS
(Depth: `references/the-clips-framework.md`.)
- **C — Choose the source:** conversational long-form clips best (~8–12 candidates/hour); non-verbal genres work
  via multimodal analysis with higher discard (test on Free first); clean audio, no burned-in subs, trim before
  upload (credits bill on source minutes).
- **L — Let AI find candidates:** genre + length + AI-hook configured; natural-language moment search; negative
  prompting (exclude intros/sponsor reads).
- **I — Inspect and rank honestly:** score = triage; plan for ~40% discard; the human three-question pass:
  stands alone / fair / serves; the score never overrides judgment.
- **P — Polish the top few:** edit only the top 3–5, approve clean mid-scorers, discard without guilt; fix the
  hook's first line, caption errors, reframe drift, boundaries; one brand template across the batch; auto-music
  OFF → licensed/native audio (or `ai-music-and-sound` / an `audio-post-mix` score/SFX plan) before export.
- **S — Ship:** download exports promptly and save them to the production folder, or file the approved picks as a
  cut list for `filmcraft` to cut locally. The drip schedule across the week is publishing (optional, user-approved,
  end of this file); recalibrating scores against real retention needs post-publish analytics (outside Creator Studio).

## The reality (verify-quarterly)
2026 OpusClip: ~10M users / ~172M clips; ClipAnything (multimodal + moment search + negative prompts); speaker
diarization layouts; ReframeAnything tracking; ~97% caption accuracy **vendor-claimed** across 20+ languages (QA
anyway); AI B-roll; XML export to Premiere/Resolve (Pro). **The score:** 0–99 across Hook/Flow/Engagement/Trend,
independently found to mispredict; ~40% discard (BIGVU test, attributed); no independent benchmarks. **Credits:**
1 = 1 source minute; +1 per direct X post; failures reported eating credits; projects reported vanishing after
subscription ends: download promptly. **Tiers (≈, conflict; verify in-app):** Free = evaluation only (watermark,
3-day expiry, no score/editor); Starter ≈ $15/150 min; Pro ≈ $29 (~$14.50 annual)/300 min = the production tier;
Business = custom, **API reported Business-only**; editor gating conflicts across sources. Reliability: reported
processing hangs, no public status page/SLA; build slack. Trustpilot 4.0 with ~22% one-star (attribute). Full
detail: `references/opus-clip-2026-reality.md`. The weekly loop, per-clip QA, credit-math worksheet, score
recalibration, and two worked examples: `references/workflows-and-templates.md`.

## In Creator Studio
- **Takes:** the finished long video (a produced final or the edited episode), trimmed to the part worth clipping.
- **Makes:** an OpusClip plan or, for picks made on the real video, a cut list (JSON below); then the reviewed
  short clips. OpusClip credits are the user's money (source minutes after trimming = credits, plus per-post
  extras): show the credit math. The human approves every clip (the three-question gate) before anything is
  exported or cut.
- **Tool:** OpusClip in the user's signed-in browser, driven by Claude (Claude in Chrome, or a headless
  agent; no OpusClip connector here, the API is reported Business-only), or approved picks go to **`filmcraft`** as
  a cut list, with no OpusClip credits spent.
- **Save:** put the plan and each clip in the production folder, write each clip's sidecar `<file>.json` (see
  `studio`) and add a line to `status.md`.

## Honest scope (never violate)
- **The agent** plans (pipeline, prompts, triage, QA, credit math) and drafts fixes; the **human reviews every
  clip** and approves (the agent can't see video; never fabricates "that clip works"). **Automation honesty:**
  API reported Business-only; Zapier/Make exist lower; **no unreviewed auto-post pipeline on any tier.** Native
  audio/stickers are added in-app by the human.
- **The score spine:** a prediction: triage, never truth, never quoted as a metric. **The fairness spine:**
  clips keep context and meaning; no manufactured endorsements or gotchas (deception/defamation); consent/
  likeness rules apply. **Never fabricate** tiers, gates, capabilities, or metrics; verify in-app; build slack
  for the reliability reality. (Full scope: `references/scope-and-connections.md`.)

## Distinct from its siblings (route correctly)
**opus-clip (this)** = the OpusClip-specific pipeline · **`captions-and-clipping`** = the general clipping craft
(read first) · **`descript`** = the long-form talk edit (master there; its Underlord clip flags overlap) ·
**`capcut`** = short-form styling/polish beyond the built-in editor · **`filmcraft`** = cuts an approved cut list
locally (reframe, captions, export) · **`narration-and-story-scripting`** = the retention craft (hooks, re-hooks)
the QA applies.

## Where this connects
Reads first: **`captions-and-clipping`** + the source's script + the project profile. Pulls the master
from: **`descript`** (the edited episode), **`clip-assemble`** / **`filmcraft`** (a produced final video),
webinars; livestream VODs (livestream-and-realtime) are outside Creator Studio. Feeds: **`capcut`** (polish),
**`filmcraft`** (cut lists). Outside Creator Studio: content-recycling, and analytics-and-reporting (per-clip
retention vs score, recalibrated monthly from native analytics; never fabricated).

## Definition of done
A clip pipeline that treats OpusClip as a first-pass finder: source chosen and trimmed for the credit mechanic
(1 credit = 1 source minute) and the spend approved by the user, candidates generated with genre/negative prompts
set, the batch triaged by score as ranking only with a planned ~40% discard, every clip passed through the human
three-question gate (stands alone / fair to the speaker with context intact / serves the audience), only the top
3–5 edited (hook line, caption accuracy on names/jargon, reframe drift, boundaries) with one brand template across
the batch and auto-music off in favor of licensed/native audio, exports downloaded promptly and registered in the
projects folder (or the approved picks filed as a cut list for `filmcraft`); tier/editor/API gating verified in-app before
subscribing (API automation claimed only at the reported Business tier; no unreviewed auto-posting anywhere);
**no score treated as truth, no out-of-context clips, no fabricated tiers/capabilities/metrics**; and correctly
distinguished from captions-and-clipping, descript, capcut and filmcraft.

## Optional: publishing
The run ends at the finished file, its sidecar `<file>.json` and `status.md`. After that, the user may choose to publish: hand the file + sidecar to a separate publishing tool if wanted, or use WoopSocial or the platform's own tools. That choice is the user's: offer it, never assume it, and take no publish action without the user's explicit OK for that post (what, where, when). The steps below are kept intact from the source plugin; WoopSocial and the platform publishing skills come from the upstream [`social-media-skills`](https://github.com/social-media-skills/skills) plugin, not from Creator Studio, so use them only if installed.

1. Batch-schedule the approved clips on a drip across the week → **scheduling-and-queue → WoopSocial**
   (WoopSocial publishes; it does not clip, edit or score video), or OpusClip's own scheduler; the choice is stated,
   not a limitation. Direct X posting from OpusClip costs +1 credit per post.
2. Platform specifics: **tiktok-video-publishing**, **instagram-reels-publishing**, **youtube-shorts**.
3. Study real retention against the scores monthly and recalibrate (native analytics; analytics-and-reporting is
   outside Creator Studio).

---

## Creator Studio: structured output

Emit the plan, or for picks made on the real video a cut list (`source` and `clips[]`
required; every `start`/`end` is a timecode in that file; `tool: "filmcraft"` when FilmCraft will cut it), and save
it as above:

```json
{"tool": "opus-clip", "source": "project artifact id or file of the long video",
 "steps": [{"id": "s1", "action": "what the human does in the tool", "notes": ""}],
 "clips": [{"id": "c1", "start": "00:12:04", "end": "00:12:51", "hook": "first line on screen", "aspect_ratio": "9:16"}],
 "captions": "style + correction pass", "gates": ["three-question review / licensing / credit checks still open"]}
```
- `references/` and `evals/` are the source plugin's, unchanged apart from a Creator Studio note; the names of
  other skills there map as in "Where this connects" above.
- Source: adapted 2026-10-08 from the [`social-media-skills`](https://github.com/social-media-skills/skills) plugin v1.0.1 (Editing & clipping category), skill
  `opus-clip` (source version 1.0.0), by Frank Heijdenrijk, MIT License (see `LICENSE` in this directory).
