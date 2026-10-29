---
name: descript
description: "Plan and run a transcript-first edit of talk content (podcasts, interviews, talking-head video) in Descript with the WORDS framework plus the interview rule (concision yes, meaning-flips never): accuracy pass, filler/silence cuts, consent-verified own-voice Overdub, Studio Sound, multicam and captions, then the episode plus human-picked clips. The agent plans and can drive Descript (Underlord) through the Descript connector only with the user's explicit OK per call; the human verifies by ear and approves; exports are registered in the projects folder. Use when someone wants to edit in Descript, edit a podcast or interview, remove filler words/silences, clean up audio, fix a flubbed word without re-recording, auto-cut between speakers, pick clips from a recording, or asks about Descript's plans, credits, or Underlord. Distinct from capcut (visual short-form), captions-and-clipping/opus-clip (clip selection at scale), filmcraft (local NLE), and speech-render/elevenlabs-voice-director (dedicated TTS)."
---

# Descript

The **talk-content editing tool skill**: write the edit in the transcript, Overdub with consent, refine the
sound, dress the visuals, and ship the cuts. The agent plans (and can drive Underlord through the Descript
connector, only with the user's explicit OK); the **human verifies by ear and approves**; the exports are
registered in the project's folder. Publishing them afterwards is optional and user-approved (see the end of
this file).

## The POV: the transcript is the timeline; decide in text, verify by ear
For dialogue-heavy content, editing the transcript beats scrubbing a timeline: delete the sentence, the clip
disappears; move the paragraph, the footage follows; reviews report ~60–70% editing-time cuts for talk content.
But the paradigm has two sharp edges the top 1% respect. **(1) The voice spine:** Overdub's consent-verified,
**own-voice-only** design is the model, not an obstacle: it exists so nobody types words into someone else's
mouth; and the craft truth is it shines on flubbed *words*, not paragraphs (long Overdub drifts synthetic;
re-record those). **(2) The meaning spine:** text-editing makes it dangerously easy to rearrange a guest into
saying something they didn't: **concision yes, meaning-flips never**, and the human owns the final cut of anyone
else's words. Operationally: the **accuracy pass is mandatory** (transcript errors become wrong edits AND wrong
captions), and since the Sept 2025 overhaul, the workflow must be **credit-aware**: media minutes count
everything you import, and formerly-unlimited AI features are metered.

## Read these first
1. The recording's script or structure: **`two-host-podcast-director`** (co-hosted podcast scripts) or
   **`narration-and-story-scripting`** (YouTube long-form, explainers and courses: its Teach / Story / Conversation
   modes). Podcast strategy itself (podcast-and-audiograms) is outside Creator Studio.
2. The project profile (`project.json`: brand constants, caption style, loudness doctrine) and its
   `workflow.md` gates. Brand voice for written outputs: the source plugin's `voice-builder` (outside Creator
   Studio) if installed.

## The framework: WORDS
(Depth: `references/the-words-framework.md`.)
- **W — Write the edit:** accuracy pass first; then cut tangents/bad takes, one-step filler+silence removal
  (Underlord; recorded talk only, never on a locked narration: "Locked narrated episodes" below), restructure by
  moving paragraphs; decide in text, **verify by ear.**
- **O — Overdub with consent:** own-voice-only, consent-verified; single words/short phrases (paragraphs =
  re-record); vocabulary/credit limits; disclose synthetic speech where required. Narration that is meant to be
  synthetic from the start belongs to **`elevenlabs-voice-director`** + **`speech-render`**, not Overdub.
- **R — Refine the sound:** Studio Sound once per source (recorded talk only, never on a locked narration: "Locked narrated episodes" below; credits; cleaner audio also improves the transcript);
  level speakers; extreme noise is a re-record, not a rescue. The final loudness pass is **`audio-master`**
  (the project's EBU R128 target), measured, not assumed.
- **D — Dress the visuals:** Automatic Multicam (record separate tracks on purpose), captions from the corrected
  transcript, human-approved B-roll, Eye Contact used honestly; beat-sync/colour route elsewhere (`capcut`,
  `filmcraft`).
- **S — Ship the cuts:** one transcript → the episode + clips (Underlord flags, the **human picks** fairly, filed
  as a cut list so `filmcraft` can cut them); show notes, chapters and a text post are packaging (outside
  Creator Studio).

## The reality (verify-quarterly)
2026 Descript: **Underlord** (agentic co-editor: filler/silence in one step, bad-take flags, B-roll suggestions,
clips, show notes) now triggerable via the **2026 public API (open beta) incl. MCP connections**; Overdub
(~24–48h training; source-audio requirements have varied, verify); Studio Sound (~10 credits/use); Automatic
Multicam; Eye Contact; ~92–95% transcription accuracy on clean audio, ~75–85% with noise/accents/jargon; ~23
languages; SOC 2 Type II; cloud-dependent (no offline). **Pricing: the Sept 2025 overhaul** moved to media
minutes + AI-credit metering of formerly-unlimited features; documented bill-shock and no mid-cycle proration
(G2, attributed); **tier figures conflict across sources, verify in-app.** Full detail:
`references/descript-2026-reality.md`. The weekly loop, credit-aware checklist, the Overdub decision table, the
interview-integrity checklist, and two worked examples: `references/workflows-and-templates.md`.

## In Creator Studio
- **Takes:** the recording or rough video/audio to edit, plus the project's rules.
- **Makes:** an edit plan for Descript or, for clips picked from the real recording, a cut list (JSON below); then
  the edited export. Descript spends the user's media minutes and AI credits (import, Studio Sound, Underlord,
  Overdub): estimate them from the plan and get the user's OK for the plan and the spend.
- **Tool:** the human edits in Descript and verifies by ear, or **the Descript connector** (`claude_ai_Descript`:
  `import_media`, `prompt_project_agent` = Underlord, `publish_project`, `export_transcript`, `export_timeline`),
  **used only with the user's explicit OK for that call**: say what it will do, to which project, and what it
  spends. Read-only calls (`list_projects`, `get_project`, `list_jobs`) still need the user to have asked for
  Descript work. No OK, no call; never a silent fallback. An approved cut list can be cut locally by **`filmcraft`**.
- **Save:** download the export (`publish_project` returns share and download URLs; this renders a file, it does not
  post anywhere) into the production folder, write its sidecar `<file>.json` (see `studio`) and add a line to
  `status.md` (audio-only episodes are mastered by
  `audio-master`).

## Honest scope (never violate)
- **The agent** plans the edit and can trigger Underlord/media actions **through the connector, only with the
  user's explicit OK** (exact human steps otherwise; no pretended automation); the **human verifies by ear**
  (pacing/tone/fairness don't live in text) and approves; the agent never fabricates "that cut sounds great."
- **Voice spine:** own-voice-only cloning; never a guest/competitor/public figure; never fabricated words in a
  real mouth; **AI-disclosure** for synthetic speech where required (EU AI Act; C2PA). **Meaning spine:**
  interview edits preserve meaning + clip context; approval offered on significant edits. **Never fabricate**
  tiers, credits, or metrics; verify in-app. (Full scope: `references/scope-and-connections.md`.)

## Locked narrated episodes (guard)
WORDS is for recorded talk. A narrated episode built from a **locked voice and its timing table** (`audio-treatment`
→ "Voice lock"; `episode-conform`) is a different job, and the talk defaults would wreck it: every hand-placed pause
and breather is a "silence", so one removal pass deletes them and shifts every cue, shot and caption after them.
- **Never** run filler removal, silence or pause removal, shortening, Studio Sound or any other processing on the
  locked voice, through Underlord or by hand. Transcript edits that move or delete words are timing changes: they
  go back to the voice lock as a new version, not into Descript.
- **Import one track per layer**, as `episode-conform` lays them out (`filmcraft` → "Assembly split"): the locked
  voice, music cues, SFX, intro and outro pieces, picture. Never a pre-mixed file as the episode audio.
- **Check the export** like any other editor's: sample rate and fps against the project, frame count, and the
  encoded-file checks (`clip-assemble` → `references/ffmpeg-recipes.md` → "Check the final encoded file"), plus the
  conform QC checklist on stems bounced from the project.
- What else must hold in any editor: `episode-conform` → "Other editors".

## Distinct from its siblings (route correctly)
**descript (this)** = text-based talk-content editing · **`capcut`** = beat-synced visual short-form (the hybrid:
master here, style cuts there) · **`captions-and-clipping` / `opus-clip`** = clip selection at scale (this feeds
them the master) · **`filmcraft`** = local NLE: transcript cuts (`transcript.removeFillers`/`removePauses`), grading,
and cutting an approved cut list without Descript credits · **`ai-voiceover`**, **`elevenlabs-voice-director` + `speech-render`** =
dedicated TTS/narration (Overdub = own-voice corrections) · **`two-host-podcast-director` /
`narration-and-story-scripting`** = the script or structure the recording follows. Outside Creator Studio:
**podcast-and-audiograms** (the strategy this tool serves), **talking-head-and-piece-to-camera** (the performance;
Eye Contact patches a read, doesn't replace delivery).

## Where this connects
Reads first: **`two-host-podcast-director`** / **`narration-and-story-scripting`** + the project profile.
Feeds: **`captions-and-clipping`** / **`opus-clip`** (the master), **`capcut`** (short-form styling),
**`filmcraft`** (cut lists), **`audio-master`** (final loudness). Outside Creator Studio: text-post-and-microblog
(the written cut), email-and-newsletter (show notes), and analytics-and-reporting (listen-through/watch-through,
read natively after release; never fabricated).

## Definition of done
A talk-content edit made at the speed of text and verified by ear: transcript corrected first (names/jargon;
errors become wrong edits and captions), tangents/bad takes cut and filler+silences removed in batched Underlord
passes (recorded talk only; a locked narration keeps every pause), restructure done in text and the result listened through; flubs fixed with consent-verified own-voice
Overdub at word/phrase length only (paragraphs re-recorded; synthetic speech disclosed where required); Studio
Sound run once per source and the final loudness measured by `audio-master`; multicam/captions/layout dressed
on-brand; the interview rule held (meaning + clip context preserved, approval offered, the human owning the final
cut of anyone else's words); one corrected transcript shipped as the episode + human-picked clips (as a cut list),
with the plan, the episode and the clips registered in the projects folder; the workflow credit-aware under the
post-Sept-2025 model (import only what you'll edit; verify tiers in-app) and every spend and connector call
approved by the user; **no cloned third-party voices, no meaning-flips, no fabricated tiers/credits/metrics**;
and correctly distinguished from capcut, captions-and-clipping/opus-clip, filmcraft and the TTS skills.

## Optional: publishing
The run ends at the finished file, its sidecar `<file>.json` and `status.md`. After that, the user may choose to publish: hand the file + sidecar to a separate publishing tool if wanted, or use WoopSocial or the platform's own tools. That choice is the user's: offer it, never assume it, and take no publish action without the user's explicit OK for that post (what, where, when). The steps below are kept intact from the source plugin; WoopSocial and the platform publishing skills come from the upstream [`social-media-skills`](https://github.com/social-media-skills/skills) plugin, not from Creator Studio, so use them only if installed.

1. Social exports → **scheduling-and-queue → WoopSocial** (WoopSocial publishes; it does not edit media).
2. Podcast audio → the RSS host (a human step).
3. YouTube → **youtube-publishing-and-metadata** (titles, chapters, description).
4. Measure listen-through/watch-through natively (analytics-and-reporting, outside Creator Studio).

---

## Creator Studio: structured output

Emit the edit plan (or, for clips picked from the real recording, a cut list, where `source` and
`clips[]` are required and every timecode is in that file) and save it as above:

```json
{"tool": "descript", "source": "project artifact id or file of the recording",
 "steps": [{"id": "s1", "action": "what the human (or an approved Underlord call) does", "notes": ""}],
 "clips": [{"id": "c1", "start": "00:12:04", "end": "00:12:51", "hook": "first line on screen", "aspect_ratio": "9:16"}],
 "captions": "style + correction pass", "gates": ["consent / meaning / credit / human review checks still open"]}
```
- `references/` and `evals/` are the source plugin's, unchanged apart from a Creator Studio note; the names of
  other skills there map as in "Where this connects" above.
- Source: adapted 2026-10-08 from the [`social-media-skills`](https://github.com/social-media-skills/skills) plugin v1.0.1 (Editing & clipping category), skill
  `descript` (source version 1.0.0), by Frank Heijdenrijk, MIT License (see `LICENSE` in this directory).
