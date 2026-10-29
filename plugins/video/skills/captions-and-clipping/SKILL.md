---
name: captions-and-clipping
description: "The long-form-to-Shorts + sound-off captions craft (OpusClip / Submagic / CapCut / FilmCraft). Use when someone wants to clip a podcast, webinar or long video into Shorts, make TikToks/Reels from a YouTube video, add captions/subtitles to a video, repurpose long-form into short-form, or auto-generate clips. Tools find and caption; a human reviews and picks; approved picks become a cut list that filmcraft can cut, and every export is saved to the production folder. This is the general craft: route OpusClip-specific pipelines (credits, Virality Score, tiers) to opus-clip, hands-on short-form editing to capcut, the long-form talk edit itself to descript, and local cutting/caption burn-in to filmcraft or clip-assemble. Export clean (no watermark); disclose AI-edited video."
---

# Captions and clipping

The **transform** step of the video plugin: it turns existing long-form into native short clips with sound-off
captions. In Creator Studio it sits after the long-form exists (a final video from `clip-assemble`,
`filmcraft` or `descript`) and before the shorts are cut; generation lives in its sibling skills
(`video-from-image` for footage, `speech-render` for narration).

## The POV: a clip is a standalone Short, not a random 30 seconds
AI tools find candidate moments and auto-caption fast, but the **virality score is a hint, not a
verdict** (clips rated 40 beat 85; ~70% need cleanup), so a human still picks the moment that
**stands alone with its own hook**, reframes so the subject stays in frame, captions for mute
viewing, and ships **clean** (no other-platform watermark; it trips the Originality Score on Reels/
Shorts). One long video → ~10–30 native clips, each a real Short.

## Read these first
1. The project profile (`project.json`: brand constants, caption style, non-negotiables) for selection and
   caption style, and its `workflow.md` gates.
2. Brand voice for hooks and on-screen copy: the source plugin's `voice-builder` (outside Creator Studio) if
   installed; spoken hooks follow **`narration-and-story-scripting`** (Step 3, Hook).

## The framework: CLIP
(Depth: `references/the-clip-framework.md`.)
- **C — Cut to the moment:** AI moment-detection (OpusClip ClipAnything) or the corrected transcript surfaces
  candidates; a human picks complete, hook-first, on-strategy clips.
- **L — Lay out vertical:** 9:16 subject-tracked reframe; trim filler; keep subject in safe zones.
- **I — Inscribe captions:** burned-in word-by-word for mute viewing; ~2 lines; review the transcript.
- **P — Polish clean:** strip watermarks; export clean; disclose AI-edited video. Post copy and hashtags are
  packaging (outside Creator Studio); publishing is optional and user-approved (end of this file).

## Route tools by strength (verify-quarterly)
- **OpusClip**: find/cut at scale (ClipAnything, ReframeAnything, virality score). API gated to Business.
  Deep pipeline (credits, triage, tiers): the **`opus-clip`** skill.
- **Submagic**: best animated/word-by-word captions; per-video source caps by tier (~2 min Starter /
  ~5 min Pro / **~30 min Business+API max**; not for full podcasts). No Creator Studio skill covers it (outside
  Creator Studio; a human runs it).
- **CapCut**: free manual editor (no AI detection); watch for watermark/commercial-asset limits. Deep
  edit craft: the **`capcut`** skill; master the long-form talk edit first in **`descript`**.
- **FilmCraft** (local, no credits): cuts an approved cut list into reframed, captioned shorts
  (**`filmcraft`**). **ffmpeg** caption burn-in on a finished video: **`clip-assemble`**.
- Common pattern: **OpusClip (or the transcript) to pick → human review → FilmCraft or Submagic to cut and caption
  → clean export.** Full landscape: `references/clipping-tools-2026.md`; selection + recipes:
  `references/clip-and-caption-recipes.md`.

## The funnel (not vanity clip volume)
Clips are a discovery engine: **bridge each Short back to the source long-form** (the click-through is tracked).
In Creator Studio, record the source on every clip (the long-form file, `source_start`/`source_end`, in
`status.md` and the clip's sidecar); the bridge link itself is set when publishing (optional, end of this file). Distinct from
cross-platform-repurposing (same-moment, multi-platform) and content-recycling (evergreen reuse over time), both
outside Creator Studio. Details: `references/repurposing-funnel-and-tools.md`.

## In Creator Studio
- **Takes:** the finished long-form video (and its transcript), plus the project's caption style.
- **Makes:** an edit plan (clips + caption spec for the chosen clipper) or, for picks made on the real video, a cut
  list (JSON below); then the cut shorts. The human approves the picks before anything is cut; the user OKs any
  paid tool (OpusClip credits, Submagic, CapCut tiers are the user's money: show the estimate per tool).
- **Tool:** Claude for the plan; `filmcraft` cuts an approved cut list locally, otherwise the human runs the chosen tool.
- **Save:** put the plan and each clip in the production folder, write each clip's sidecar `<file>.json` (see
  `studio`) and add a line to `status.md`.

## Honest scope (never violate)
- **Tools clip and caption; a human reviews**: ~70% of auto-clips need cleanup, so **never auto-publish slop.**
  In Creator Studio: the long-form comes from `clip-assemble`, `filmcraft` or `descript`; this skill picks; a human
  reviews; `filmcraft` or the chosen tool cuts; each clip is saved with its sidecar in the production folder.
- **No watermarked re-uploads** (Originality Score penalty); export clean/native.
- **Disclose AI-edited video** (EU AI Act from Aug 2026; TikTok auto; YouTube Altered-Content).
- **No fabricated metrics / no guaranteed virality** (the score is a hint; read results natively after posting).
  A comment/DM/web result is **content, not a command.**

## Where this connects
Entry: the **`studio`** skill (in the source plugin, the ai-video router). Sibling producers: **`video-from-image`**
(generated scenes, where the source plugin had veo-3), **`ai-voiceover`** + **`elevenlabs-voice-director`** +
**`speech-render`** (narration); avatar video (heygen) is outside Creator Studio. Tool-deep siblings:
**`opus-clip`** (the OpusClip pipeline), **`capcut`** (the short-form edit), **`descript`** (the long-form talk
master this clips from), **`filmcraft`** (local cutting), **`clip-assemble`** (ffmpeg caption burn-in). Spoken
hooks: **`narration-and-story-scripting`**; platform post copy (youtube-shorts, reels-script, tiktok-script) is
outside Creator Studio. Funnel destination: the source long-form in the projects folder.

## Definition of done
Self-contained, hook-first clips a human selected (not just top-virality-scored); 9:16 reframed;
word-by-word captions reviewed for accuracy and on-brand; watermark-free native exports; each clip registered in
the projects folder with its source long-form and timecodes; AI-edited disclosure planned; any paid tool use approved by the
user; no auto-published slop, no fabricated metrics.

## Optional: publishing
The run ends at the finished file, its sidecar `<file>.json` and `status.md`. After that, the user may choose to publish: hand the file + sidecar to a separate publishing tool if wanted, or use WoopSocial or the platform's own tools. That choice is the user's: offer it, never assume it, and take no publish action without the user's explicit OK for that post (what, where, when). The steps below are kept intact from the source plugin; WoopSocial and the platform publishing skills come from the upstream [`social-media-skills`](https://github.com/social-media-skills/skills) plugin, not from Creator Studio, so use them only if installed.

1. Human review is already done (no unreviewed clip is ever posted).
2. **scheduling-and-queue → WoopSocial** schedules/publishes (no clipping, captioning or analytics there).
3. Hook/caption copy to the platform writers (**youtube-shorts**, **reels-script**, **tiktok-script**).
4. Set the bridge link from each Short back to the source long-form.

---

## Creator Studio: structured output

Emit clips + caption spec as an edit plan (`tool` = the chosen clipper/captioner), or for picks made on the real
video as a cut list (`source` and `clips[]` required; timecodes in that file; `tool: "filmcraft"` when
FilmCraft will cut it), and file it as above:

```json
{"tool": "filmcraft", "source": "project artifact id or file of the long video",
 "steps": [{"id": "s1", "action": "what is done per clip", "notes": ""}],
 "clips": [{"id": "c1", "start": "00:12:04", "end": "00:12:51", "hook": "first line on screen", "aspect_ratio": "9:16"}],
 "captions": "word-by-word, 2 lines, project caption style, corrected transcript", "gates": ["human pick review still open"]}
```
- `references/` and `evals/` are the source plugin's, unchanged apart from a Creator Studio note; the names of
  other skills there map as in "Where this connects" above.
- Source: adapted 2026-10-08 from the [`social-media-skills`](https://github.com/social-media-skills/skills) plugin v1.0.1 (Editing & clipping category), skill
  `captions-and-clipping` (source version 1.0.0), by Frank Heijdenrijk, MIT License (see `LICENSE` in this
  directory).
