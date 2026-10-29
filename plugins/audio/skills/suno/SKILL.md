---
name: suno
description: "Suno music craft (TRACK framework), rights first: arrange a music brief for Suno (style prompt, meta-tag structure, exclude-styles, lyrics, vocal spec) and generate it in the signed-in app on a paid plan. Use for brand themes, podcast intros/outros, jingles, consistent sonic identity (Personas, own Voice, Custom Models), stems, or \"do I own my Suno songs\". Web route only; the user's ears judge every take; paid step needs approval."
---

# Suno

The Suno lane of the audio plugin: target the brief, rights first, arrange with structure, curate and humanize,
keep disclosure clean. This skill arranges the brief; **Claude drives the signed-in Suno app in the user's browser** (Claude in Chrome, or
a headless browser) and generates from the checklist; **the user's ears judge the keeper**; the download is persisted to the projects folder and mixed by `audio-post-mix`.
Adapted from `social-media-skills:suno` (Frank Heijdenrijk, MIT; see `LICENSE`).

## The POV: a full song from a sentence, but the rights are the craft
Suno v5.5 makes the most natural vocals in the category, splits its own generations into clean stems, and offers a
consistency stack (Personas, your own verified Voice, a Custom Model trained on tracks you own). But 2026 Suno is
defined by its legal moment, so work rights-first. **(1) Paid plan before the first note:** free-tier songs are
non-commercial **forever**; upgrading never unlocks them. **(2) Commercial rights are not ownership:** post-Warner
terms grant paid users commercial rights, while Suno's own docs concede purely-AI music isn't copyrightable; human
elements (original lyrics, a re-recorded part over stems, real DAW arrangement) strengthen the claim and reclassify
the work as AI-assisted. **(3) No indemnification while the lawsuits run:** Warner settled and partnered (Nov 2025);
UMG and Sony are still suing (July 2026 fair-use hearing), so favour generic style language over artist
references and never clone an artist's voice. **(4) Archive now:** pre-deal models face deprecation; export WAVs,
stems and MIDI. And: **trending audio is native-only**; Suno is the owned brand sound, not the trend sound.

## Read these first
1. **`ai-music-and-sound`**: the model-agnostic brief, source pick and licence craft (it writes the
   music brief this skill arranges).
2. **The project** (`project.json` brand) and what the music serves: the narration
   (`narration-and-story-scripting`) or the show (`two-host-podcast-director`). The source's `brand-profile`,
   `short-form-video-script` and `podcast-and-audiograms` are outside Creator Studio.

## The framework: TRACK
(Depth: `references/the-track-framework.md`.)
- **T - Target the brief:** genre (1-2) + one mood anchor + 2-4 named instruments + vocal spec (or instrumental).
  Music does a job: brief it like a director.
- **R - Rights first:** paid tier before generating; commercial rights are not copyright; no indemnification;
  generic styles over artist names; never a cloned artist voice.
- **A - Arrange with structure** (`arrange_track`): Custom mode + meta-tags ([Intro]/[Verse]/[Chorus]/[Bridge]/
  [Outro]), exclude-styles, short verse blocks (~4 lines), Extend from a timestamp; the consistency stack (Persona,
  own Voice, Custom Model; test for reported regressions).
- **C - Curate and humanize:** variations, then the user's ears pick the seed; human elements for anything that
  matters; archive WAVs/stems/MIDI before deprecation; batch stem jobs (failed regenerations reportedly still cost
  credits).
- **K - Keep disclosure clean:** record the AI origin in the production manifest; DDEX AI flag and no PRO
  registration for pure-AI works if the user chooses to distribute (see "Optional: publishing").

## The reality (verify-quarterly)
2026 Suno: **v5.5** (Mar 2026), **Studio** (Premier DAW: 12 zero-bleed stems, MIDI export), Personas, **Voices**
(own voice, verified), **Custom Models** (6+ owned tracks), hum-to-song, an API (Python/Node; Zapier/Make; tier
gating conflicts across sources). Legal spine: RIAA suit (2024), **Warner settled + partnered Nov 2025**
(commercial rights; free commercial downloads removed; pre-deal model deprecation scheduled; artist opt-in
announced), **UMG + Sony still suing; July 2026 hearing**; GEMA verdict mid-2026. No retroactive rights; no
indemnification; purely-AI output not copyrightable; DDEX AI-disclosure enforced since late 2025. Tiers about Free
50 credits/day (non-commercial), Pro about $8-10 (2,500 credits, commercial rights), Premier about $24-30
(10,000 + Studio; same rights); credits don't roll over. These figures are in `studio` skill → `references/models/suno.json`
(provisional). Known frictions (community-reported): Persona/Voice regressions; Studio credit burns on failed stem
jobs; fully-AI tracks underperform human releases on saves ~25-40% (one firm's data). Full detail:
`references/suno-2026-reality.md`; brief pattern, use-case router, archive checklist and worked examples:
`references/brief-patterns-and-templates.md`.

## In Creator Studio
- **Takes:** the music brief from `ai-music-and-sound` (`source: suno`); `studio` skill → `references/models/suno.json` is only a
  provisional lookup.
- **Makes:** the arranged brief (style prompt, exclude-styles, structure, lyrics; below), then the downloaded takes
  (WAV, plus stems/MIDI for keepers). The agent cannot hear: it records every take, says which one the mix used, and
  the **user picks the keeper** when they listen (record a selected-take swap and refresh the affected mix/review).
- **Tool:** Claude (free) for the arrangement; the Suno web app in the signed-in browser on a **paid plan** for
  generation (read the cost on the Create button; generate once; Extend rather than re-roll).
- **Save:** put the takes in the production folder and add a line to `status.md`. Then the `ai-music-and-sound` licence check, `audio-post-mix`, `audio-master`.

## Honest scope (never violate)
- This skill writes briefs, lyrics, meta-tag structures, exclude-styles, the consistency plan and the archive
  plan. Generation is a **web step the user runs**; no Suno API is called (no MCP here; the API's tier
  gating is unverified). No pretended automation.
- **The user's ears judge every track**: never claim "that sounds great".
- **Rights spine:** paid tier first; no retroactive rights; commercial rights are not ownership; human elements for
  anything that matters; no indemnification, so litigation-aware prompting (generic styles); counsel for
  high-stakes work (not legal advice). **Voice/likeness:** never clone a real artist or fake a collab.
  **Never fabricate** tiers, legal status or performance claims. (Full scope: `references/scope-and-connections.md`.)

## Distinct from its siblings
**suno (this)** = the Suno lane · **ai-music-and-sound** = the source-agnostic brief + licence craft (read first)
· **sound-render** = ElevenLabs Music/SFX renders (the safer-licence default) · **ai-voiceover** /
**speech-render** = spoken TTS (Suno's Voices is *singing*, own-voice-only) · **capcut** / **descript** (video
plugin) = where a track scores or sits under an edit · **two-host-podcast-director** = the show an intro/outro sting
serves · outside Creator Studio: `luma` / `ai-video` (the visuals in the source plugin), `podcast-and-audiograms`.

## Optional: publishing
The production run ends at the finished file, its sidecar `<file>.json` and `status.md`. Publishing is an
**optional** step after that: if the user wants it, hand the file + sidecar to a separate publishing tool. It runs only if the
user chooses it, and each publishing action needs the user's explicit approval (the same rule as a paid step). The
source's publishing skills (`social-media-skills` plugin, if installed) are outside Creator Studio.
- **Distribution** (streaming services): flag the AI origin with the DDEX AI flag via the distributor; no PRO
  registration (ASCAP/BMI) for pure-AI works; honest performance expectations; free-tier tracks can never be
  distributed.
- **Social:** the finished content -> `scheduling-and-queue` -> **WoopSocial** (it does not generate, attach or
  license audio). Trending audio is native-only (in-app). Measuring performance is the source's
  `analytics-and-reporting` (outside Creator Studio): never fabricated.

## Definition of done
A track briefed with director-grade specificity, arranged with meta-tags and grown by Extend, generated on a paid
plan once authorized, picked by the user's ears, human elements added where it matters, keepers archived
(WAV + stems + MIDI) and persisted to the projects folder, licence and AI origin recorded in the manifest; litigation-aware
throughout (generic styles, no artist cloning or fake collabs, no indemnification assumed); no fabricated tiers,
legal claims or capabilities.

---

## Creator Studio: structured output
The arranged brief is the music brief with `source: "suno"` and the Suno fields filled:
```json
{"job": "series theme, 15/30/60 s cuts", "source": "suno", "genre": ["soft jazz"], "mood": "quietly reassuring",
 "instruments": ["warm upright piano", "brushed drums"], "vocals": "instrumental", "tempo": "~85 BPM",
 "prompt": "soft jazz. Mood: quietly reassuring. Instruments: warm upright piano, brushed drums. Instrumental. ~85 BPM",
 "structure": ["Intro", "Verse", "Outro"], "exclude_styles": ["vocals", "heavy drums"],
 "license": {"tier": "Pro or Premier (paid)", "archive": "WAV + stems + MIDI"}}
```
Generation is a web checklist; the downloaded take is saved to the production folder (`asset-persist`).
- Source: copied 2026-10-08 from the [`social-media-skills`](https://github.com/social-media-skills/skills) plugin v1.0.1, skill `suno`, by Frank Heijdenrijk,
  MIT License (see `LICENSE` in this directory). Adapted to Creator Studio; craft method kept.
