# ElevenLabs 2026 capabilities — legacy orientation

> **Creator Studio note (2026-10-08):** adapted from social-media-skills v1.0.1 (Frank Heijdenrijk, MIT; see
> `../LICENSE`). Craft text kept; cross-references point at Creator Studio skills or say "outside Creator
> Studio"; tool-file pointers removed. The production pipeline ends at the QC'd master + manifest; publishing
> steps (WoopSocial and similar) are OPTIONAL, run only if the user chooses them and approves (SKILL.md
> "Optional: publishing").
> Skills named here that Creator Studio does not ship are optional: use them only if the upstream
> [`social-media-skills`](https://github.com/social-media-skills/skills) plugin is installed; otherwise the project
> file and Creator Studio's own skills cover their role.

*Legacy source claims below are [unverified] unless labelled otherwise. Confirm model IDs, route controls and prices before rendering. Current cited tagging/model guidance: the `studio` skill → `references/elevenlabs-tagging.md`.*

## What it is
The leading AI voice platform: text-to-speech, voice cloning, dubbing, sound effects, music, and
voice agents — one shared credit pool. Output is genuinely hard to distinguish from a human read
at normal listening speed. **It generates audio; it does not edit/mix it** (no built-in post).
PlayHT shut down Dec 2025; ElevenLabs and Murf are the dominant premium platforms; ElevenLabs leads
on quality, cloning, and languages.

## Models (pick by job)
- [verified] **Eleven v4 / v3** — shared bracket Audio Tag syntax for emotional direction (e.g. `[whispers]`, `[excited]`, `[laughs]`); v4 90+ languages / 10,000 chars, v3 70+ / 5,000. v4 is ElevenLabs' recommendation for new work, never a code default; selected v3 stays valid. Source: shared tagging reference above. [maintainer-observed] Delivery is model-specific; listening review rules are in that reference.
- **Multilingual v2** — stable, lifelike, best for **long-form/final** production; 29+ languages;
  192kbps (Creator+). Slower but most consistent over long passages.
- **Flash v2.5** — ultra-low latency (~75ms), 32 languages, ~0.5 credit/char. *Drafts, real-time, testing.*
- **Turbo v2.5** — quality/speed balance, 32 languages.
- Model IDs: `eleven_v3`, `eleven_multilingual_v2`, `eleven_flash_v2_5` (also `eleven_turbo_v2_5`, `eleven_v4`:
  `studio` skill → `references/models/elevenlabs.json`). In Creator Studio the model comes from the project voice profile.

## Voices
- **Voice Library** (10,000+), **Voice Design** (design a voice from a prompt).
- **Instant Voice Cloning (IVC)** — from **1–2 min of clean audio** (don't exceed ~3 min);
  Starter+; fast approximation; can wobble on long/unusual passages.
- **Professional Voice Cloning (PVC)** — longer sample, trained model, studio-quality, holds over
  long-form; Creator+. **ElevenLabs verifies identity and currently only lets you clone your OWN
  voice via PVC.** Cloned voices speak 32+ languages even from a one-language sample (may carry the
  source accent).

## Delivery controls (the craft levers)
[inferred] The numerical tuning ranges below are practitioner starting points, not vendor defaults or v3/v4 mode prescriptions; preserve the profile and check route support. Verified defaults and caveats are in the shared tagging reference.
- **Stability:** 0.3–0.5 = more expressive/varied; 0.7–1.0 = more consistent.
- **Similarity Boost:** 0.75–0.85 recommended; 1.0 can introduce artifacts from the training audio.
- [verified] **Audio Tags** (v3/v4) for emotion/pacing; no SSML `<break>`. `<phoneme>` only on Flash v2. [unverified] v3/v4 pronunciation-dictionary behavior. See shared tagging reference.

## Adjacent tools
**Dubbing / Dubbing Studio** (preserve the speaker's voice across 70+ languages; non-English quality
uneven); **Studio** (long-form podcasts/audiobooks editor); **Scribe v2** STT (diarization,
timestamps); **Eleven Music** (licensed, commercial-safe); Sound Effects; Voice Changer/Isolator;
Agents Platform (real-time voice bots — mostly dev-facing).

## Pricing + credits (verify-quarterly)
1 char = 1 credit (Multilingual v2/v3); Flash/Turbo ~0.5/char; one shared pool; roll over up to
2 months; API draws the same pool (no extra charge), list ~$0.10/1k chars (v2/v3), $0.05 (Flash).
- Free: ~10k credits (~10 min); **no commercial license; must attribute ElevenLabs.**
- Starter $6: ~30k credits; commercial rights; **IVC**. Creator $22: ~121k; **PVC**; 192kbps.
- Pro $99: ~600k; 44.1kHz PCM via API. Higher tiers (Scale/Business) for volume + org-wide PVC.

## Ethics (enforced by this skill)
Clone your own voice freely; **cloning anyone else needs documented consent**; no celebrity
soundalikes for commercial; fraud/impersonation is illegal in most jurisdictions; disclose AI
voice in ads/political content. ElevenLabs' policy explicitly bars unauthorized replication.
