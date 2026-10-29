# Scripting for the ear + two worked VO scripts

> **Creator Studio note (2026-10-08):** adapted from social-media-skills v1.0.1 (Frank Heijdenrijk, MIT; see
> `../LICENSE`). Craft text kept; cross-references point at Creator Studio skills or say "outside Creator
> Studio"; tool-file pointers removed. The production pipeline ends at the QC'd master + manifest; publishing
> steps (WoopSocial and similar) are OPTIONAL, run only if the user chooses them and approves (SKILL.md
> "Optional: publishing").
> Skills named here that Creator Studio does not ship are optional: use them only if the upstream
> [`social-media-skills`](https://github.com/social-media-skills/skills) plugin is installed; otherwise the project
> file and Creator Studio's own skills cover their role.

## Write for the ear (the 80%)
- Contractions, short sentences, spoken rhythm. "You'll" not "you will."
- One idea per breath; vary sentence length for a natural cadence.
- Spell tricky names/jargon phonetically in a pronunciation note.
- **Read it aloud first.** If you stumble, rewrite the line.
- Don't paste eye-copy (blog paragraphs, bullet lists) — rewrite it as speech.

## Direct the delivery
- **Model:** use the resolved project `voices[].model`; v4 is recommended for new work, selected v3 remains valid. Never a code default or automatic draft/final swap.
- **Audio Tags (v3/v4):** sparing emotion/pacing cues, e.g. `[warm]`, `[whispers]`, a beat `...`.
- **Settings** ([inferred] practitioner starting points, not vendor defaults; verify route support): Stability ~0.3–0.5 (expressive) vs ~0.7–1.0 (consistent); Similarity ~0.75–0.85.
- Compare authorized takes; keep `generations_count: 1` per approved render and request a retake only for selected weak lines.

## Model-by-job, fast
| Job | Model | Why |
|---|---|---|
| Reel/Short VO (expressive) | Selected v3/v4 | Shared tag syntax, model-specific delivery ([maintainer-observed]); see shared tagging reference; v4 recommended for new work |
| Explainer / course / audiobook | Multilingual v2 | stable over long-form |
| Draft / internal / real-time | Flash v2.5 | ~half credits, ~75ms |
| Localized versions | Dubbing | preserves voice, 70+ langs |

## Worked example 1 — Reel VO (blunt indie-founder voice, ~18s, Eleven v3)
```
VOICE: designed "dry, low-key founder"   MODEL: eleven_v3   STABILITY: 0.4
SCRIPT (for the ear, tags sparing):
"Almost missed four grand in tax this year. [beat] Not proud of it.
I was tracking income. I wasn't tracking the set-aside.
So now? Thirty percent of every invoice, separate account, same day it lands.
[warm] Boring. Works. Saved my April."
CAPTIONS: burned in (mute viewing). DISCLOSURE: "AI voiceover" in caption.
```

## Worked example 2 — explainer narration (warm studio voice, ~30s, Multilingual v2)
```
VOICE: consented founder clone (PVC, own voice)   MODEL: eleven_multilingual_v2   STABILITY: 0.6
SCRIPT:
"If tax season makes your stomach drop -- this'll help.
We do one thing, and the whole panic just... goes.
Open a second account. Call it Tax.
Then move thirty percent the moment a client pays you.
You never see it, so you never spend it -- and April stops hurting."
LOCALIZE: Dubbing into ES/FR/DE, founder voice preserved; review each.
CAPTIONS: burned in. DISCLOSURE: "AI voiceover (our founder's voice)".
```

Both: read the project's brand + voice constants first (the source's brand-profile + voice-builder are
outside Creator Studio); written for the ear; directed (model/tags/settings);
captioned for mute viewing; consent + disclosure handled. ElevenLabs generates the audio (speech-render); audio-post-mix / audio-master mix and master it (ElevenLabs has no
built-in editor). B-roll → the video plugin (`video-from-image`; veo-3 / ai-video are outside Creator Studio); avatars → heygen
(outside Creator Studio); captions/clipping → captions-and-clipping; optional publish (user approves) →
scheduling-and-queue → WoopSocial.

Current cited tag support, limits and route controls: the `studio` skill → `references/elevenlabs-tagging.md`. Examples require the project profile to select the shown model; legacy price/latency/localization figures here are [unverified].
