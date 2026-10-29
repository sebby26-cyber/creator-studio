> **Creator Studio note (2026-10-08):** copied unchanged from the `social-media-skills` plugin v1.0.1 (MIT, see
> `../LICENSE`). In Creator Studio, publishing steps named here (WoopSocial, scheduling-and-queue, the platform
> publishing skills) are optional, after the master and only with the user's OK (`../SKILL.md` "Optional:
> publishing"); `tools/integrations/*.md` and `tools/REGISTRY.md` are not vendored; every
> other skill named here maps to a Creator Studio skill or is marked outside Creator Studio in `../SKILL.md`
> ("Where this connects" and "Distinct from its siblings").
> Skills named here that Creator Studio does not ship are optional: use them only if the upstream
> [`social-media-skills`](https://github.com/social-media-skills/skills) plugin is installed; otherwise the project
> file and Creator Studio's own skills cover their role.

# The CLIP framework — turn a long video into real Shorts

A clip is **not** a random 30 seconds of a long video — it's a **self-contained Short with its own
hook.** The tools find candidates and auto-caption, but the virality score is a hint, not a verdict
(rated-40 clips beat rated-85; ~70% need cleanup). CLIP keeps a human in the loop where it matters.

## C — Cut to the moment
- Use AI moment-detection (Opus Clip ClipAnything) to surface candidates fast, including
  natural-language search ("find where I talk about pricing").
- **Then choose with judgment:** a clip needs a complete thought and a hook in the first 1–3 seconds.
  Don't ship "loud moments" the AI flagged that don't stand alone. Accuracy drops on multi-speaker
  content — review those harder.
- Align selection to strategy and the brand (brand-profile), not raw clip count.

## L — Lay out vertical
- Auto-reframe to **9:16** with subject tracking (ReframeAnything); fix any frame where the speaker
  drifts out. Keep faces/subjects in the safe zone, clear of the platform UI.
- Trim dead air, filler, and false starts so the clip is tight from frame one.

## I — Inscribe captions (for mute viewing)
- **Burn in word-by-word / karaoke captions** — most viewers are on mute; captions lift completion,
  add accessibility, and get indexed for search.
- ~2 lines per frame, inside safe zones; **review the transcript** for names/jargon (auto-captions
  are ~97–99% but mis-hear specifics). Submagic for the best animated styling.
- Keep the caption style on-brand (brand-profile), not a generic viral template.

## P — Polish & publish clean
- **Strip every other-platform watermark.** A TikTok/CapCut watermark trips the Originality Score on
  Reels/Shorts and throttles reach. Export clean and native.
- Hand the **hook + caption copy** to the platform writer (youtube-shorts / reels-script /
  tiktok-script) and the publish step to **scheduling-and-queue → WoopSocial.**
- **Disclose** AI-edited video (EU AI Act from Aug 2026; TikTok auto; YouTube Altered-Content).
- Bridge each clip back to the **source long-form** (the Short→long click-through is tracked).

## The clip brief a request should fill
```
SOURCE: long video → target platforms (9:16)
SELECT: candidate moments (AI) → human-picked self-contained clips w/ hooks
REFRAME: 9:16 subject-tracked; trim filler
CAPTIONS: word-by-word, on-brand, transcript reviewed
CLEAN: no watermark; native export
HANDOFF: hook/caption → platform writer; publish → scheduling-and-queue → WoopSocial
DISCLOSURE: AI-edited label
```
