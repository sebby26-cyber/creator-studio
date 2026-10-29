---
name: capcut
description: "Plan a short-form edit (TikTok, Reels, Shorts) in CapCut with the EDITS framework: retention-paced cuts to the script's beats, auto-captions with a mandatory accuracy pass, beat-synced and license-checked sound, export to spec. The agent plans and flags the licensing/ToS gates; the human executes in CapCut and approves; the export is registered in the projects folder. Use when someone wants to edit in CapCut, cut a reel/short, set up auto-captions, use CapCut's AI tools, pick a plan, or asks whether CapCut music/assets or client work are safe commercially. Safety spine: client/NDA work edits locally, never synced (the ToS licenses uploads); never pirated assets, non-consensual lip-sync, or uncorrected captions. Distinct from narration-and-story-scripting (the script), captions-and-clipping/opus-clip (clipping long video), descript (long-form), filmcraft (local NLE) and video-from-image (generation)."
---

# CapCut

The **short-form editing tool skill**: edit to the script, drive the pace, ink the captions, tighten sound with
the license check, and ship safe and to spec. The agent plans and flags the hazards; the **human executes and
approves**; the finished export is registered in the project's folder. Publishing it afterwards is optional and user-approved
(see the end of this file).

## The POV: the edit serves the script, and "free" has fine print
CapCut earned its place as the default short-form editor: auto-captions in seconds, beat-sync, background removal,
trend templates, phone-to-desktop sync, with a **~15-minute ceiling by design** (it's a short-form tool; route
long-form to `descript` or `filmcraft`). Two top-1% edges most creators learn the hard way. **(1) The licensing fine
print:** the June 2025 ToS grants ByteDance a perpetual, transferable, sublicensable license to anything
**uploaded/synced** (you keep copyright; it's a license, not ownership, stated precisely), so **client/NDA
material edits locally, never synced**; and the music trap: free-library assets are personal-use only, and even
tracks labeled **"Commercial use" are reported cleared for TikTok/CapCut platforms only**; YouTube/IG claims
still happen, verify per track. **(2) Check the tier before you build:** features migrate behind Pro over time,
and the documented failure is hours of editing ending at a paywalled export. The craft line: the edit **executes
what the script designed** (`narration-and-story-scripting`, social-shorts delivery): pacing, captions and sound
serve the beats; and auto-captions always get an accuracy pass, because a wrong claim in a caption is an error
that ships with the file.

## Read these first
1. The script: the narration script from **`narration-and-story-scripting`** (the beats this edit executes).
2. The project profile (`<projects>/projects/<slug>/project.json`: brand constants, caption style, loudness and
   timing doctrine) and its `workflow.md` gates. Brand voice for any on-screen copy: the source plugin's
   `voice-builder` (outside Creator Studio) if installed.

## The framework: EDITS
(Depth: `references/the-edits-framework.md`.)
- **E — Edit to the script:** load the beats; cut everything that isn't in them; the hook lands in the first
  seconds; the last cut is the loop.
- **D — Drive the pace:** visual change every ~2–4s (jump cuts, punch-ins/keyframes, B-roll, text pops);
  beat-sync; scrub-test for dead stretches.
- **I — Ink the captions:** auto-captions + the mandatory accuracy pass; style once (the project's caption
  style), save the preset; sound-off first; SRT/VTT export when copy travels.
- **T — Tighten sound + effects (the license gate):** license-check music BEFORE export (the "Commercial
  use"-is-TikTok-only trap); platform-native audio, licensed tracks, a music brief from **`ai-music-and-sound`** (rights-checked), or a score/SFX
  plan from **`audio-post-mix`**;
  narration from **`speech-render`**; effects sparingly.
- **S — Ship safe + to spec:** right resolution/fps per destination; ToS hygiene (local-only for client work);
  likeness/AI-disclosure handled; export, then register the file in the projects folder (below).

## The reality (verify-quarterly)
2026 CapCut: auto-captions (free ~10-min/project cap; Pro unlimited; SRT/VTT/ASS export), real-time background
removal, motion/camera tracking, vocal isolation + flicker removal (Pro), TTS/avatars, Long-Video-to-Shorts,
keyframes/beat-sync, MP4/MOV to 4K (Pro). Plans restructured early 2026: **Standard ≈ $9.99 / Pro ≈ $19.99
(~$15 annual); sources conflict (regional/legacy $7.99–9.99 figures), verify in-app**, and features migrate
behind Pro over time (check tiers before building). Documented billing/cancellation complaints (manage via app
store). The ToS + music-license spines as above. India ban since 2020. **Attribute all; verify-quarterly.** Full
detail: `references/capcut-2026-reality.md`. The daily loop, the pre-edit tier check, the client-work hygiene
checklist, long-form routing, and two worked examples: `references/workflows-and-templates.md`.

## In Creator Studio
- **Takes:** the script beats and the source footage/audio to edit, plus the project's caption and brand rules.
- **Makes:** an edit plan for CapCut (JSON below) with the tier it needs (Pro-only features, export resolution);
  then the human's exported video. The user OKs the plan, the tier and every open licensing/ToS gate first.
- **Tool:** Claude for the plan; the human edits in CapCut (there is no CapCut connector; the agent never claims to
  see the cut). A plain cut list with no CapCut-only effects can instead go to **`filmcraft`** (local, automated).
- **Save:** put the plan and the export in the production folder, write the export's sidecar `<file>.json` (see
  `studio`) and add a line to `status.md`.

## Honest scope (never violate)
- **The agent** plans the edit (cut plan, pacing, caption spec, sound plan + license gates, export spec) and
  flags every ToS/licensing hazard; the **human executes in CapCut**, judges footage, approves the cut (no CapCut
  agent connector; never pretend to see or edit video; never fabricate "that cut works").
- **Licensing/ToS spine:** personal-use free assets; "Commercial use" music verified per track; uploaded-content
  license → **client/NDA work local-only, never synced** (counsel for high-stakes; not legal advice); users bear
  responsibility for uploads. **Likeness + AI-disclosure:** no non-consensual lip-sync/deepfakes; disclose
  TTS/AI voices where required (EU AI Act; C2PA). **Never** pirated packs, unlicensed music, uncorrected
  captions, or fabricated prices/gates/metrics. (Full scope: `references/scope-and-connections.md`.)

## Distinct from its siblings (route correctly)
**capcut (this)** = the short-form timeline edit · **`narration-and-story-scripting`** = designs the beats (read
first) · **`captions-and-clipping` / `opus-clip`** = clip long video into shorts (feeds this) · **`descript`** =
text-based talk/long-form editing (the >15-min route) · **`filmcraft`** = local NLE: finishing a timeline, or
cutting an approved cut list automatically · **`clip-assemble`** = ffmpeg assembly and caption burn-in ·
**`video-from-image`** / **`image-generate`** = generate footage and stills (this edits them) · **`speech-render`**
= narration audio. Outside Creator Studio: **canva** (graphics + simple motion), **talking-head-and-piece-to-camera**
(the on-camera performance).

## Where this connects
Reads first: **`narration-and-story-scripting`** + the project profile. Pulls footage from:
**`video-from-image`** (generated, disclosed), **`captions-and-clipping`** / **`opus-clip`** (clips), and on-camera
recordings (talking-head-and-piece-to-camera, outside Creator Studio). Hands off: the registered export, via the
`studio` production manifest. Outside Creator Studio: before-after-and-transformation, content-recycling, and
analytics-and-reporting (watch-through / 3s hold, read natively after posting; never fabricated).

## Definition of done
A short-form edit that executes the script's beats (hook inside the first seconds, dead air gone, the loop cut
last), paced with a visual change every ~2–4s and beat-synced sound, captioned via auto-captions plus the
mandatory accuracy pass in a saved on-brand preset inside safe zones, with every asset license-checked before
export (personal-use free assets avoided for commercial work; "Commercial use" music verified per destination
platform; no pirated packs or unlicensed songs), the ToS hygiene held (client/NDA material local-only, never
synced), likeness/AI-disclosure handled (no non-consensual lip-sync; TTS/avatars disclosed where required), and
the export to spec per destination; tiers checked and approved before the edit was built (no paywall surprise at
export), the human executing and approving the cut, and the plan plus the export registered in the projects folder;
**no fabricated prices, feature gates, capabilities, or metrics**; and correctly distinguished from
narration-and-story-scripting, captions-and-clipping/opus-clip, descript, filmcraft and the video generators.

## Optional: publishing
The run ends at the finished file, its sidecar `<file>.json` and `status.md`. After that, the user may choose to publish: hand the file + sidecar to a separate publishing tool if wanted, or use WoopSocial or the platform's own tools. That choice is the user's: offer it, never assume it, and take no publish action without the user's explicit OK for that post (what, where, when). The steps below are kept intact from the source plugin; WoopSocial and the platform publishing skills come from the upstream [`social-media-skills`](https://github.com/social-media-skills/skills) plugin, not from Creator Studio, so use them only if installed.

1. Export the finished file to spec per destination.
2. **scheduling-and-queue → WoopSocial** publishes it (WoopSocial validates TikTok's required fields; it does not
   edit video). CapCut's direct-to-TikTok path exists too; which path is used is a stated choice, not a limitation.
3. Platform specifics: **tiktok-video-publishing**, **instagram-reels-publishing**, **youtube-shorts**. Native-only
   features (trending audio, some stickers) are added in-app by the human on either path.
4. Measure natively after posting (analytics-and-reporting, outside Creator Studio); never fabricated.

---

## Creator Studio: structured output

Emit the edit plan as JSON and save it as above:

```json
{"tool": "capcut", "source": "what is being edited (project artifact id, file, project)",
 "steps": [{"id": "s1", "action": "what the human does in the tool", "notes": ""}],
 "clips": [{"id": "c1", "start": "00:00:00", "end": "00:00:41", "hook": "first line on screen", "aspect_ratio": "9:16"}],
 "captions": "style + correction pass", "gates": ["licensing / consent / tier / human review checks still open"]}
```
- The human executes in CapCut and approves; nothing automated runs this plan. It is not a cut list:
  a CapCut plan is one paced edit (punch-ins, beat-sync, text pops), not a list of standalone shorts, so
  `filmcraft`'s shorts cutting would cut it wrongly.
- `references/` and `evals/` are the source plugin's, unchanged apart from a Creator Studio note; the names of
  other skills there map as in "Where this connects" above.
- Source: adapted 2026-10-08 from the [`social-media-skills`](https://github.com/social-media-skills/skills) plugin v1.0.1 (Editing & clipping category), skill
  `capcut` (source version 1.0.0), by Frank Heijdenrijk, MIT License (see `LICENSE` in this directory).
