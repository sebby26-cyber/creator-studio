> **Creator Studio note (2026-10-08):** copied unchanged from the `social-media-skills` plugin v1.0.1 (MIT, see
> `../LICENSE`). In Creator Studio, publishing steps named here (WoopSocial, scheduling-and-queue, the platform
> publishing skills) are optional, after the master and only with the user's OK (`../SKILL.md` "Optional:
> publishing"); `tools/integrations/*.md` and `tools/REGISTRY.md` are not vendored; every
> other skill named here maps to a Creator Studio skill or is marked outside Creator Studio in `../SKILL.md`
> ("Where this connects" and "Distinct from its siblings").
> Skills named here that Creator Studio does not ship are optional: use them only if the upstream
> [`social-media-skills`](https://github.com/social-media-skills/skills) plugin is installed; otherwise the project
> file and Creator Studio's own skills cover their role.

# Scope, distinctions & connections

## Honest scope (never violate)
- **The agent** plans the edit — the cut plan against the script's beats, pacing/keyframe notes, caption style
  spec, sound plan with the license gates, export spec — and flags every ToS/licensing hazard. There is no
  CapCut agent connector: the **human executes in CapCut**, judges the footage, and approves the cut (the agent
  never pretends to see or edit video, and never fabricates "that cut works").
- **WoopSocial publishes** the finished export (TikTok required fields validate at that layer) (measurement: the platforms' native
  analytics). It does **not** edit video, add native trending audio, or judge a cut. CapCut's direct-to-TikTok
  publish path exists — the WoopSocial split (10 platforms, validation, one queue) is a stated stack choice, not
  a fabricated limitation.
- **The licensing spine:** free-library assets are personal-use only; "Commercial use" music is reported cleared
  for TikTok/CapCut platforms only (**verify per track**); users are responsible for what they upload. **The ToS
  spine:** uploaded/synced content grants ByteDance a perpetual, transferable, sublicensable license (+ name/
  likeness for sponsored identification) — copyright stays yours; state it precisely, don't fearmonger; **client/
  NDA material edits locally, never synced** (high-stakes → counsel; not legal advice).
- **Likeness + AI-disclosure:** never lip-sync/deepfake a real person without consent (real or AI lookalike);
  disclose TTS/AI-avatar voices where platform/region requires (EU AI Act; C2PA). **Never** use pirated packs or
  unlicensed music. **Never fabricate** a price, feature gate, or metric — verify in-app (verify-quarterly).

## Distinct from its siblings (route correctly)
- **short-form-video-script** = designs the beats this edit executes (read first; the edit serves the script).
- **captions-and-clipping / opus-clip** = clipping long video into shorts (feeds this for styling/pace).
- **descript** (Wave 12) = text-based editing for talk/long-form (the >15-min route; hybrid pipeline partner).
- **canva** = graphics + simple motion (Video 2.0); **this** = the timeline short-form edit.
- **ai-video / veo-3 / kling / runway** = *generate* footage; **this** edits it.
- **talking-head-and-piece-to-camera** = the on-camera performance this footage comes from.
- **tiktok-video-publishing / instagram-reels-publishing / youtube-shorts** = the platform publish specifics.

## Where this connects
- **Reads first:** short-form-video-script (the beats) + brand-profile/design-and-templates (caption style, type,
  color) + voice-builder (any written copy/TTS).
- **Pulls footage from:** talking-head-and-piece-to-camera (recorded), ai-video/heygen (generated, disclosed),
  captions-and-clipping (long-form cutdowns).
- **Feeds:** tiktok-video-publishing, instagram-reels-publishing, youtube-shorts (the finished file),
  before-after-and-transformation (the reveal edit), content-recycling.
- **Publishes via:** export → scheduling-and-queue → **WoopSocial.**
- **Tool file:** `tools/integrations/capcut.md` (plans, license gates, ToS hygiene).
- **Measure with:** native + analytics-and-reporting on watch-through/3s hold — never fabricated.
