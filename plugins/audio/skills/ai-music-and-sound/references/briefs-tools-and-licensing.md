# Brief, tools, licensing + two worked examples

> **Creator Studio note (2026-10-08):** adapted from social-media-skills v1.0.1 (Frank Heijdenrijk, MIT; see
> `../LICENSE`). Craft text kept; cross-references point at Creator Studio skills or say "outside Creator
> Studio"; tool-file pointers removed. The production pipeline ends at the QC'd master + manifest; publishing
> steps (WoopSocial and similar) are OPTIONAL, run only if the user chooses them and approves (SKILL.md
> "Optional: publishing").
> Skills named here that Creator Studio does not ship are optional: use them only if the upstream
> [`social-media-skills`](https://github.com/social-media-skills/skills) plugin is installed; otherwise the project
> file and Creator Studio's own skills cover their role.

## The music brief (mood/genre/tempo to the video)
```
VIDEO: length, # of cuts, the beats (hook @0-3s, build, payoff/CTA)
FEEL: genre + mood + energy curve (e.g. "warm lo-fi, low energy, gentle lift at the payoff")
TEMPO: BPM band or "matches the cut rhythm"; where it swells/drops to the edit
REFERENCE: a vibe/era to aim at -- NOT a copyrighted song to clone
VOCALS: instrumental vs vocal (vocals can fight a voiceover -> coordinate with ai-voiceover)
```

## Sound-design accents (often > the track for short-form)
```
Whoosh on a hard cut | riser into the payoff | stinger on the logo/CTA | subtle ambience under a talking head
ElevenLabs SFX or a library SFX pack. Keep it tasteful -- accents punctuate the edit, they don't bury it.
```

## Tool-pick decision (verify-quarterly)
| Need | Pick |
|---|---|
| Safest commercial / brand work | **ElevenLabs Music + SFX** (licensed from day one) → **sound-render** |
| Simple, indemnified, huge catalog | **stock library** (Epidemic / Artlist / Soundstripe) |
| Best raw quality, accept some risk | Suno v5 / Udio v2 (**paid**, licensed 2026 models; moderate Content ID risk) → **suno** (Udio: outside Creator Studio) |
| Voice / narration | → **ai-voiceover** + **elevenlabs-voice-director** / **speech-render** (ElevenLabs) |
| The video tool already makes audio | native video-tool audio, e.g. Veo (the source's **ai-video**: outside Creator Studio) |

## Licensing / Content ID / disclosure checklist
```
[ ] Paid tier (commercial rights) -- NOT a free tier
[ ] License/receipt saved on file; human contribution documented
[ ] Not copyrighted or platform trending music (for a brand) without a license
[ ] AI-disclosure applied per platform/region (EU AI Act from Aug 2026; C2PA)
[ ] Accept Content ID can still claim AI audio -- have the license ready to dispute
```

## Creator Studio flow (+ optional WoopSocial publishing)
```
ai-music-and-sound briefs music + sound design + picks the safest licensed source (the music brief)
-> sound-render (ElevenLabs) / suno (web) renders it once authorized, or the user licenses a library track
-> asset-persist into the production folder; the licence check records licence + disclosure
-> audio-post-mix mixes it under the voice; audio-master masters it; file + sidecar + status.md (the run ends here)
-> OPTIONAL, only if the user chooses and approves: WoopSocial publishes the finished video (audio baked in). WoopSocial does NOT generate music,
   add native trending audio (not via API), clear licenses, or run Content ID.
```

## Worked example 1 - product Reel (blunt indie-founder voice)
```
30s product Reel, 5 cuts. Brief: warm minimal electronic, low energy, small lift at the payoff. Source: ElevenLabs
Music, paid -- cleanest license, no Suno/Udio baggage. Sound design: one whoosh on the mid cut, a soft stinger on the
logo. I keep the license receipt. It is mixed under the voice and mastered; if I choose and approve, WoopSocial posts the finished video. No trending TikTok sound on a brand ad.
```

## Worked example 2 - recipe series (warm studio voice)
```
A cozy recipe series -- we want a gentle, consistent bed. We use a stock library (Artlist) for an indemnified instrumental,
plus a tiny "sizzle" SFX on the cooking shot. It's instrumental so it sits under the voiceover (ai-voiceover) cleanly. We
note the AI/disclosure where required, save the license, and mix it in; publishing via WoopSocial is optional -- one small, safe step at a time.
```

Both: brief to the video + energy curve; safest licensed source (ElevenLabs/library, paid); tasteful sound design; license
kept + disclosure; mixed + mastered in Creator Studio; optional WoopSocial publishing on the user's OK; never copyrighted/trending music for a brand.
