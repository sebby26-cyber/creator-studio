# Scope, distinctions + connections

> **Creator Studio note (2026-10-08):** adapted from social-media-skills v1.0.1 (Frank Heijdenrijk, MIT; see
> `../LICENSE`). Craft text kept; cross-references point at Creator Studio skills or say "outside Creator
> Studio"; tool-file pointers removed. The production pipeline ends at the QC'd master + manifest; publishing
> steps (WoopSocial and similar) are OPTIONAL, run only if the user chooses them and approves (SKILL.md
> "Optional: publishing").
> Skills named here that Creator Studio does not ship are optional: use them only if the upstream
> [`social-media-skills`](https://github.com/social-media-skills/skills) plugin is installed; otherwise the project
> file and Creator Studio's own skills cover their role.

ai-music-and-sound briefs **original/licensed audio beds + sound design** for social video. The agent
**briefs the music + sound design + picks the safest licensed source + advises licensing/Content-ID/
disclosure**; the **tool generates/licenses** the audio; **`audio-post-mix`** mixes it under the voice and the pipeline ends at
the master; optionally, if the user chooses and approves, WoopSocial publishes the finished video (outside Creator Studio).

## Honest scope (never violate)
- **The agent** briefs the **music + sound design** (mood/genre/tempo to the video; SFX/transition accents),
  **picks the safest licensed source** (ElevenLabs Music/SFX or a stock library over the contested Suno/Udio;
  **paid tier** for commercial rights), and **advises** licensing / Content ID / AI-disclosure.
- **The tool** generates or licenses the audio (ElevenLabs / Suno / Udio / Stable Audio, or Epidemic /
  Artlist / Soundstripe). **`audio-post-mix`** mixes the audio + SFX under the voice; **`audio-master`** masters it.
- **Optional, user's choice + approval: WoopSocial publishes the finished video** (audio baked in). It does **NOT** generate music, add native
  **trending audio** (not via API), clear licenses, or run **Content ID**.
- **Never copyrighted/trending music for a brand without a license** (mute/claim/legal); **paid tier for
  commercial rights** (free ≠ commercial); **pure AI music may not be copyrightable**; **indemnification
  mostly absent** (ElevenLabs cleanest); **keep the license + document human input**; **AI-disclosure**;
  **never promise "100% legally safe"** (litigation is live). **Verify-quarterly.**

## Distinct from its siblings
- **ai-music-and-sound (this)** — the **music + sound design** (the audio **bed** under a video).
- **ai-voiceover** — the **voice / narration** (ElevenLabs). **They pair** (both audio; both can use
  ElevenLabs; coordinate so music doesn't fight the voice).
- **captions-and-clipping** (video plugin) — the **captions / clip cutdowns** (the on-screen text + the edit).
- **narration-and-story-scripting** — the narration this sits under (the source's **reels-script** /
  **tiktok-script** are outside Creator Studio).
- **ai-video** (outside Creator Studio) — **native video-tool audio** (e.g. Veo generates audio with the clip); this is a **dedicated
  music/SFX layer** added separately.
- **the platform publishing skills** (`instagram-reels-publishing`, `tiktok-video-publishing`; outside Creator
  Studio, optional publishing) — where
  **native trending audio** lives (in-app); this is the **original/licensed** alternative (brand-safe).

## Where this connects
- **Reads first:** the project's brand constants (the source's `brand-profile` is outside Creator Studio),
  the **narration / cue list it scores** (the narration script and `audio-post-mix` cue list: length, beats).
- **Pairs with:** `ai-voiceover` (voice + music together), `audio-post-mix` (the cue map + the mix), `captions-and-clipping` (captions over the same video), native video-tool audio when enough (outside Creator Studio).
- **Tools:** ElevenLabs Music/SFX (`sound-render`), Suno (`suno`), Udio, Stable Audio + stock libraries
  (Epidemic/Artlist/Soundstripe; licensed by the user). Prices/ids: `studio` skill → `references/models/elevenlabs.json`,
  `studio` skill → `references/models/suno.json`.
- **Ends at:** the QC'd master + manifest. **Optional publishing** (user's choice + approval):
  `scheduling-and-queue → WoopSocial` (the finished video; outside Creator Studio). **Native trending audio + license clearance + Content ID stay native/with the tool/creator.**
