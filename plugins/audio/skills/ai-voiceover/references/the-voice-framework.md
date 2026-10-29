# The VOICE framework — 80% script + direction, 20% tool

> **Creator Studio note (2026-10-08):** adapted from social-media-skills v1.0.1 (Frank Heijdenrijk, MIT; see
> `../LICENSE`). Craft text kept; cross-references point at Creator Studio skills or say "outside Creator
> Studio"; tool-file pointers removed. The production pipeline ends at the QC'd master + manifest; publishing
> steps (WoopSocial and similar) are OPTIONAL, run only if the user chooses them and approves (SKILL.md
> "Optional: publishing").
> Skills named here that Creator Studio does not ship are optional: use them only if the upstream
> [`social-media-skills`](https://github.com/social-media-skills/skills) plugin is installed; otherwise the project
> file and Creator Studio's own skills cover their role.

Most AI voiceover sounds robotic for two reasons: people feed it text written for the **eye**, and
they accept the **default read**. A great AI VO is ~20% the tool and ~80% the script-for-the-ear and
the direction. VOICE keeps it human — with consent and disclosure as the spine.

## V — Voice match
- Pick from the **library**, **Voice Design** a voice from a prompt, or use a **consented clone**
  (your own voice via IVC/PVC). Match the brand (the project; the source's brand-profile is outside Creator Studio) and the platform.
  In Creator Studio the voice is locked in `project.json` voices[] (cast via elevenlabs-voice-director).
- **The brand's *written* voice (the source's voice-builder, outside Creator Studio; here the project's brand
  constants) is matched by an *audio* voice + delivery.** Keep them consistent.

## O — Own the script for the ear
- Write to be **heard, not read**: spoken cadence, contractions, short sentences, one idea per breath.
- **Read it aloud** — if you stumble, the model will too. Kill tongue-twisters and run-ons.
- Front-load the point; no eye-only formatting (the model can't see your bold/bullets).

## I — Inflect & direct
- **Model:** use the resolved project `voices[].model`; v4 is recommended by ElevenLabs for new work, selected v3 stays valid. Both support **Audio Tag syntax**; delivery is model-specific ([maintainer-observed]; see the shared tagging reference); never silently switch models for drafts/finals.
- **Direct the delivery:** Audio Tags for emotion/pacing, deliberate pauses and emphasis,
  pronunciation/phonetics for names and jargon.
- **Settings** ([inferred] practitioner starting points, not vendor defaults; verify route support): Stability ~0.3–0.5 for expressive reads, ~0.7–1.0 for consistency; Similarity Boost
  ~0.75–0.85. Compare authorized takes; no guarantee of identical output.

## C — Caption alongside
- **Most social video is watched on mute.** The VO **supports** captions; it doesn't carry the video
  alone. Plan burned-in captions (captions-and-clipping) and make the visuals legible silent.
- **Localize** with Dubbing (70+ languages, voice preserved) — review each language; quality is uneven.

## E — Ethics: consent + disclosure
- **Only consented voices:** your own clone, a consented person, a library/designed voice, or
  licensed talent. **Never** clone a real person (celebrity, competitor, narrator) without documented
  consent — PVC verification only permits your own voice anyway.
- **Disclose** AI voice where it matters (EU AI Act; TikTok auto-disclosure; especially ads/political).

## The brief a VO request should fill
```
VOICE: library | designed | consented clone — character + why it fits the brand
MODEL: project voices[].model (v3/v4 for tags; no code default)
SCRIPT (for the ear): spoken cadence, tags for emotion/pauses, pronunciation notes
SETTINGS: Stability / Similarity
CAPTIONS + LOCALIZATION: sound-off plan; languages (Dubbing)
ETHICS: consent confirmed; disclosure plan
```

Current cited tag support, limits and route controls: the `studio` skill → `references/elevenlabs-tagging.md`. Examples require the project profile to select the shown model; legacy price/latency/localization figures here are [unverified].
