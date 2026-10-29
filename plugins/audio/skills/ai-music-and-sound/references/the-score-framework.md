# The SCORE framework — score a social video with safe, licensed audio

> **Creator Studio note (2026-10-08):** adapted from social-media-skills v1.0.1 (Frank Heijdenrijk, MIT; see
> `../LICENSE`). Craft text kept; cross-references point at Creator Studio skills or say "outside Creator
> Studio"; tool-file pointers removed. The production pipeline ends at the QC'd master + manifest; publishing
> steps (WoopSocial and similar) are OPTIONAL, run only if the user chooses them and approves (SKILL.md
> "Optional: publishing").
> Skills named here that Creator Studio does not ship are optional: use them only if the upstream
> [`social-media-skills`](https://github.com/social-media-skills/skills) plugin is installed; otherwise the project
> file and Creator Studio's own skills cover their role.

Most social audio is a **background bed + sound design** under a short video, not a released song. SCORE
briefs it safely. The agent **briefs the music + sound design + picks the safest licensed source + advises
licensing**; the **tool generates/licenses** the audio; **`audio-post-mix`** mixes it under the voice and the pipeline ends at the master; optionally, if the user chooses and approves, WoopSocial publishes the finished video (outside Creator Studio).

## S — Set the brief to the video
- **Mood / genre / tempo / energy** matched to the video's **beats + structure** (what the track does at the
  **hook / build / payoff**). Brief the feeling + reference vibe, not a copyrighted song.

## C — Choose the safest licensed source
- **ElevenLabs Music/SFX** (cleanest, licensed-from-day-one) or a **stock library** (Epidemic / Artlist /
  Soundstripe) **over the contested Suno/Udio.** **Paid tier for commercial rights** (free ≠ commercial).

## O — Orchestrate the sound design
- **SFX, whooshes, risers, stingers on the cuts** — for short-form this **often matters more than the
  track.** (ElevenLabs SFX / library SFX.)

## R — Respect the license + Content ID
- **Paid tier = commercial** (free ≠); **pure AI music may not be copyrightable**; **indemnification mostly
  absent** (ElevenLabs cleanest); **Content ID can claim even AI audio** → keep the license + **document
  human input**; **AI-disclosure**; **NEVER copyrighted/trending music for a brand without a license** (mute/
  claim/legal).

## E — Embed it (+ optional publish)
- **`audio-post-mix`** places music + SFX under the voice at the plan's levels and **`audio-master`**
  masters it (audio-only work; in a video, every layer goes on its own track and is mixed in the editor: `filmcraft` → "Assembly split"). Optional, user's choice and
  approval: **WoopSocial publishes the finished video** — it doesn't **generate music** or add native **trending
  audio** (not via API; risky for brands).

## The audio brief a request should fill
```
SET: mood/genre/tempo/energy to the video's beats (hook/build/payoff); reference vibe, not a copyrighted song
SOURCE: ElevenLabs Music/SFX or a stock library (Epidemic/Artlist/Soundstripe) > Suno/Udio; PAID tier (commercial rights)
SOUND DESIGN: SFX/whoosh/riser/stinger on the cuts (often > the track for short-form); listed in the cue list's cues[].sfx
LICENSE: paid=commercial (free isn't); pure AI may not be copyrightable; indemnification mostly absent; keep license + document human input; AI-disclosure
EMBED: audio-post-mix mixes it under the voice -> audio-master -> manifest; OPTIONAL (user approves): WoopSocial publishes the finished video (no native trending audio via API). NEVER copyrighted/trending music for a brand. Never promise "100% safe".
```
