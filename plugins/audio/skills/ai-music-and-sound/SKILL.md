---
name: ai-music-and-sound
description: "Music and sound design for a production (SCORE framework): brief the music bed and SFX to the program's beats, pick the safest licensed source (ElevenLabs Music/SFX or a stock library over Suno/Udio; paid tier), and check licence, Content ID and AI-disclosure. Use for \"background music\", \"score this\", \"sound effects\", \"which AI music tool is safe\" or \"can I use this trending song\". Writes the music brief; rendering is sound-render or suno (paid, approval first)."
---

# AI music and sound

The music and sound-design brief for a production: the **audio bed and SFX under the voice**. The brief here is
"audio that won't get muted, claimed, or sued". This skill briefs the music, picks the safest licensed source and
checks the licence; `sound-render` (ElevenLabs Music/SFX) or `suno` renders it; `audio-post-mix` mixes it under
the voice and `audio-master` masters it. Adapted from `social-media-skills:ai-music-and-sound` (Frank Heijdenrijk,
MIT; see `LICENSE`).

## The POV: the brief is "audio that won't get muted, claimed, or sued"
Generative music is a working tool (Suno, Udio, ElevenLabs Music, Stable Audio, Lyria, MusicGen), but for a brand
or creator the question is not "does it sound good" but **"will this trigger a Content ID claim, a platform mute,
or legal exposure?"** So pick the **safest licensed source** (ElevenLabs Music/SFX or a stock library over the
contested Suno/Udio), use a **paid tier** for commercial rights, and **never use copyrighted or trending music for
a brand without a licence.**

## Read these first
1. **The project** (`project.json` brand + `audio.mix_levels_db`): the feel and the levels. (The source's
   `brand-profile` is outside Creator Studio; the project file plays its role here.)
2. **What the music scores:** the narration script and, when planned, the cue list from `audio-post-mix`
   (beats, timing, feeling, music and SFX per cue).

## The framework: SCORE
(Depth: `references/the-score-framework.md`.)
- **S - Set the brief to the program** (`brief_music`): mood, genre, tempo and energy to the beats (hook, build,
  payoff). When a cue list exists, use its direction and measured placements: a continuous bed, evolving music, separate cues or silence; effects may be empty. Reference a vibe, never a
  copyrighted song.
- **C - Choose the safest licensed source:** ElevenLabs Music/SFX (`source: elevenlabs`, rendered by
  `sound-render`) or a stock library (`source: library`; the user licenses it, no render step) over Suno/Udio.
  Suno (`source: suno`) when the user wants it; its rights spine is in the `suno` skill. Paid tier for
  commercial rights.
- **O - Orchestrate the sound design:** SFX, whooshes, risers, stingers on the cuts (for short-form this often
  matters more than the track). The SFX list lives in the cue list (`cues[].sfx`, written by
  `audio-post-mix`); apply this guidance there. `sound-render` renders it.
- **R - Respect the licence and Content ID** (`license_check`): paid = commercial (free is not); pure AI music
  may not be copyrightable; indemnification mostly absent (ElevenLabs cleanest); Content ID can claim even AI
  audio, so keep the licence and document human input; AI-disclosure.
- **E - Embed it:** `audio-post-mix` plans the music and SFX under the voice. An audio-only program is mixed
  there and mastered by `audio-master`; an episode with picture is mixed in the editor with every layer on its own
  track (`filmcraft` → "Assembly split"). Publishing the finished video is optional (see "Optional: publishing").

## Picking the music (shortlist, genre fit, credit lines)
This is the one home of music selection; other skills point here.
- **Research the genre first.** Before picking, look at what the genre's reference channels or shows actually put under their narration (bed style, tempo, energy, how often it changes, where it stops) and write two or three lines of findings into the brief. Reference a vibe, never a copyrighted track to clone.
- **The bed serves the genre's feeling,** which the project sets; write it into the brief. [observed in one science-mystery documentary] For mystery and science-mystery narration the bed that worked created suspense, curiosity or a sense of adventure (pulses, low drones, evolving textures, restrained tension); calm, mellow or "relaxing" beds, and a symphonic or piano feel outside true adventure moments (a launch, a journey, a discovery payoff), worked against it. Treat that as a starting point for the genre, not a rule for every project.
- **Keep a shortlist file** in the production folder (for example `music-shortlist.md`): one row per candidate with title, author or source, link or provider job, licence (exact name and version, for example CC BY 4.0, a library licence, a paid AI tier), the **exact credit line** the licence requires (copied from the source, not paraphrased), where it must appear (video description, end credits), the receipt location, which cue it is for, and keep or reject with the reason. Candidates are source assets: never delete them (`asset-persist` → "Cleanup: keepsake rule").
- **Credits travel forward.** A CC BY track needs its credit in the video description; without it, and sometimes even with it, Content ID may claim the video, so the credit and licence must be ready. The selected tracks' credit lines go into the cue bindings (`audio-post-mix`) and the final file's sidecar `credits` (`studio` → `references/output-sidecar.md`), so a publishing step can paste them.

## The reality (verify-quarterly)
RIAA sued Suno and Udio (June 2024); Warner settled with Suno (Nov 2025); UMG settled with Udio (Oct 2025;
royalty ~$0.002-0.005, licensed 2026 platform); Sony still litigating both; a fair-use ruling around summer 2026
could set precedent; indie class actions pending. **ElevenLabs Music (Aug 2025) is the safest** (licensed from day
one); Suno best quality with moderate Content ID risk; Udio moderate risk; Stable Audio / Lyria / MusicGen
licensed; stock libraries (Epidemic, Artlist, Soundstripe) clean and indemnified. Full detail:
`references/ai-music-2026-reality.md`; the brief, sound-design accents, tool pick, licence checklist and worked
examples: `references/briefs-tools-and-licensing.md`.

## In Creator Studio
- **Takes:** the project's sound intent and levels, the narration script, and the measured narration timing from
  `audio-post-mix` once it exists (provisional script timing is replaced before placement).
- **Makes:** a music brief (below; continuous music, separate cues or silence as the story needs; effects may be
  empty) with the source pick the user confirms, the music shortlist with licence and exact credit line per
  candidate, and a licence check on each kept track.
- **Tool:** Claude (free). Renders are `sound-render` (ElevenLabs; `estimate_only` first: SFX 40 credits per second,
  music priced by the preflight) or `suno` (web, Claude in the browser; read the cost on Create); `generations_count: 1`
  unless the user asks for takes.
- **Save:** put the brief, the kept tracks and the licence + disclosure record in the production folder and add a
  line to `status.md`.

## Honest scope (never violate)
- **This skill** briefs the music and sound design, picks the safest licensed source, and advises on licence,
  Content ID and AI-disclosure. **The tool** generates or licenses the audio. **The mix** happens in
  `audio-post-mix` (local ffmpeg). The user's ears judge every take at the listen gate.
- **Never copyrighted or trending music for a brand without a licence**; paid tier = commercial (free is not);
  pure AI music may not be copyrightable; indemnification mostly absent; keep the licence and document human
  input; AI-disclosure; **never promise "100% legally safe."** (Scope detail:
  `references/scope-and-connections.md`.)

## Distinct from its siblings
**ai-music-and-sound (this)** = the music + sound-design brief and licence check · **suno** = the Suno-specific
lane (TRACK framework) · **sound-render** = ElevenLabs Music/SFX rendering · **audio-post-mix** = cue map, levels
and the mix · **ai-voiceover** / **elevenlabs-voice-director** = the voice (they pair; instrumental beds sit under
narration cleanly) · **captions-and-clipping**, **capcut** = captions and the edit (video plugin) · outside
Creator Studio: `reels-script` / `tiktok-script` (social scripts; narration here is
`narration-and-story-scripting`), `ai-video` (native video-tool audio).

## Optional: publishing
The production run ends at the finished file, its sidecar `<file>.json` and `status.md`. Publishing is an
**optional** step after that: if the user wants it, hand the file + sidecar to a separate publishing tool. It runs only if the
user chooses it, and each publishing action needs the user's explicit approval (the same rule as a paid step). The
source's publishing skills (`social-media-skills` plugin, if installed) are outside Creator Studio.
- Source flow, if the user opts in: the finished video (audio baked in) -> `scheduling-and-queue` ->
  **WoopSocial**. WoopSocial does NOT generate music, add native trending audio (not via API), clear licences, or
  run Content ID.
- Native platform **trending audio** is in-app only (`instagram-reels-publishing` / `tiktok-video-publishing`
  in the source plugin) and legally risky for a brand; offer an original licensed track with the same vibe instead.
- Apply the AI-disclosure recorded in the manifest at upload; keep the licence receipt ready for a Content ID
  dispute.

## Definition of done
Music and SFX briefed to the program's beats; the safest licensed source chosen on a paid tier; every paid render
priced by preflight and approved; outputs persisted to the projects folder; the licence saved, human contribution
documented and the disclosure plan in the manifest, every kept track's exact credit line in the shortlist and
handed forward to the sidecar `credits`, with the honest caveats stated; no copyrighted or trending
music for a brand without a licence; nothing fabricated.

---

## Creator Studio: structured output
The music brief:
```json
{"job": "score the narration under the voice", "source": "elevenlabs",
 "genre": ["ambient orchestral"], "mood": "quiet wonder", "instruments": ["low strings", "soft piano"],
 "vocals": "instrumental", "tempo": "~70 BPM, follows the beats",
 "cues": [{"beat": "b1", "start_s": 0.5, "duration_s": 12, "prompt": "ambient orchestral, quiet wonder, low energy with a gentle lift at the payoff, instrumental under narration"}],
 "license": {"tier": "paid", "receipt": "where the licence is saved", "disclosure": "AI music label where required"},
 "notes": ["vocals would fight the voiceover: instrumental"]}
```
The licence check is a QC report with `checklist: "music-licence"` (items: paid tier, receipt on file, human
input documented, no copyrighted/trending music, disclosure plan) and an overall `verdict`.
- Source: copied 2026-10-08 from the [`social-media-skills`](https://github.com/social-media-skills/skills) plugin v1.0.1, skill `ai-music-and-sound`, by Frank
  Heijdenrijk, MIT License (see `LICENSE` in this directory). Adapted to Creator Studio; craft method kept.
