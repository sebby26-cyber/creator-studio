# Scope, distinctions & connections

> **Creator Studio note (2026-10-08):** adapted from social-media-skills v1.0.1 (Frank Heijdenrijk, MIT; see
> `../LICENSE`). Craft text kept; cross-references point at Creator Studio skills or say "outside Creator
> Studio"; tool-file pointers removed. The production pipeline ends at the QC'd master + manifest; publishing
> steps (WoopSocial and similar) are OPTIONAL, run only if the user chooses them and approves (SKILL.md
> "Optional: publishing").
> Skills named here that Creator Studio does not ship are optional: use them only if the upstream
> [`social-media-skills`](https://github.com/social-media-skills/skills) plugin is installed; otherwise the project
> file and Creator Studio's own skills cover their role.

## Honest scope (never violate)
- **The agent** writes the briefs, lyrics, meta-tag structures, exclude-styles, the consistency plan
  (Persona/Voice/Custom Model), the archive plan, and — **where an API key/connection exists** — can drive
  generation programmatically (Python/Node libraries; Zapier/Make; API tier gating conflicts across sources —
  verify). Where it doesn't, exact steps for the human in-app. No pretended automation.
- **The human's ears judge every track** — the agent can't hear audio and never fabricates "that sounds great";
  curation is the human's craft.
- **Optional** (user's choice + approval): **WoopSocial publishes** the content the music sits under; it does **not** generate, attach, or license
  audio. Trending audio remains **native-only** in the platform apps.
- **The rights spine:** paid plan before generating anything commercial; free-tier tracks non-commercial
  forever (no retroactive rights); **commercial rights ≠ ownership/copyright** (purely-AI output isn't
  copyrightable; human elements strengthen claims and reclassify as AI-assisted); **no indemnification** —
  litigation is live (UMG/Sony; July 2026 hearing), so favor generic style language over artist references;
  high-stakes → counsel (not legal advice).
- **Voice/likeness:** never clone a real artist's voice or imply involvement (fake collabs = fraud); Suno's
  own-voice verification and the Warner artist **opt-in** model are the consent standard. **Disclosure:** DDEX
  AI flag at distribution; no PRO registration for pure-AI works; **AI-disclosure** on content where
  platform/region requires (EU AI Act). **Never fabricate** tier terms, credit rules, legal status, or
  performance claims — verify-quarterly.

## Distinct from its siblings (route correctly)
- **ai-music-and-sound** = the model-agnostic music/audio router + licensing craft (read it first; this is the
  Suno-specific lane).
- **ai-voiceover / elevenlabs-voice-director / speech-render** = spoken voice/TTS (narration, VO) — Suno's Voices is *singing*, own-voice-only.
- **capcut** (video plugin) = where the track scores the video (and whose library-music license trap a Suno brand track
  replaces); **descript** (video plugin) = talk-content editing (its audio is speech, not music).
- **two-host-podcast-director** = the show an intro/outro sting serves (the source's **podcast-and-audiograms**
  is outside Creator Studio).
- **luma / ai-video** (outside Creator Studio) = the visuals the music pairs with (Luma outputs silent video — this is the other half).
- **brand-profile / voice-builder** (outside Creator Studio; here the project's brand constants) = the brand
  feel the sonic identity translates.

## Where this connects
- **Reads first:** ai-music-and-sound (router + licensing) + the project (the feel) + what the music serves (`narration-and-story-scripting` /
  `two-host-podcast-director`; the source's short-form-video-script / podcast-and-audiograms are outside Creator Studio).
- **Feeds:** `audio-post-mix` (mix_scored, under the narration), `capcut` (scoring an edit), the show's
  intro/outro. Outside Creator Studio: luma/ai-video sequences, tiktok-video-publishing /
  instagram-reels-publishing / youtube-shorts (optional publishing).
- **Ends at:** the QC'd master + manifest. **Optional publishing** (user's choice + approval): the finished
  video/content → scheduling-and-queue → **WoopSocial** (outside Creator Studio).
- **Config:** `studio` skill → `references/models/suno.json` (model label, provisional credit figure, web route).
- **Measure with:** human-ears fitness + native performance of the scored content (the source's analytics-and-reporting, outside Creator Studio) — never fabricated.
