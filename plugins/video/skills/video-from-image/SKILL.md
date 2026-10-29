---
name: video-from-image
description: "Video through OpenRouter, or image-to-video, start/end-frame and motion-transfer clips on Higgsfield under the explicit production model/route/seed/quality choices (Seedance, Kling, Veo, Genjutsu...). Identity rides on the start frame or an Element (no video model takes a Soul id). Prompts written with the `higgsfield` skill (MCSLA)."
---

# Video from image

- **Takes:** selected start/end frames (or a subject still plus a motion reference video), and the model, route, seed and quality choices.
- **Makes:** video clips, one per shot.
- **Tool:** Higgsfield (connector, CLI or web) or OpenRouter (curl); Higgsfield prompts written with the `higgsfield` skill.
- **Save:** put the clips in the production folder and add a line to `status.md`.

1. **Inputs:** `studio` supplies project/production identity, exact selected start/end frames or motion reference, model/route/seed and draft/final settings. The model is the user's named model for this scope, else the production's recorded choice, else the project's `models.video.default` (respecting `allowed` and `locked`; `models.motion.default` for motion transfer), else recommend one from the selected provider’s model reference (`studio` → `references/models/higgsfield.json` or `references/models/openrouter.json`) with the reason and record it. Defaults for aspect, duration and resolution come from `defaults.video.*`. Confirm aspect ratio, media roles and supported controls with specialist guidance/current tool schema. Reuse provider job references when supported; upload local inputs when required.
For OpenRouter, continue with the section below; the Higgsfield prompt, presets and motion-transfer instructions apply only to Higgsfield. Resolve `routes.openrouter` (route `openrouter`) and preserve selected frames; never silently turn an image-to-video request into text-to-video.

2. **Prompt:** write it with the `higgsfield` skill (MCSLA structure for video, named presets and params from its
   files, never invented), as Creator Studio requires for every Higgsfield prompt.
3. **Preflight:** check exact route cost and existing authorization/project caps. Do not add a universal per-clip gate or batch-cap requirement. Explain unsupported model/seed/reference controls before generating; do not substitute.
4. **Generate, wait, persist** (`asset-persist`), then QC per the project.

## Style preset ("viral")
A clip can take an optional style preset. A request that says "viral" ("<character> image turned into a viral clip")
asks for a viral preset. Resolve the actual preset: the Higgsfield catalog changes, so use
**resolve via `mcp__higgsfield__get_presets` with `{source: "viral"}` at execute time** (read-only browsing). Show
the user the presets that fit an image-to-video clip of this subject; the user picks one (browsing never
authorizes `execute_preset`); read its `get_preset_instructions` and follow them, with its own cost preflight and
OK. If the user declines every preset, use the selected `generate_video` route under existing authorization. A named preset other than viral must
come from the `higgsfield` skill's files (`skills/higgsfield-motion/SKILL.md`), never invented.
When no start frame is supplied, generate/select the still first with `image-generate`, then make the clip.

## Motion transfer
Takes a subject still and the user's reference video; makes one clip. Plain
English: "motion swap", "motion transfer/control", "copy the moves", "reference video". The user names the
video's path in the request (`/path/ref.mp4`) or it is recorded in `status.md`. A character alone is rendered to a still
first (Genjutsu takes reference images, not a Soul id).
- Model `hf_mult_motion_control` (Higgsfield Genjutsu) through `mcp__higgsfield__generate_video`, per the
  Higgsfield MCP's own routing (never the legacy `motion_control` tool for a Genjutsu edit). The catalog lists only
  `resolution` (480p / 720p / 1080p, default 720p): no prompt, duration or aspect ratio is sent.
- Media roles: subject still(s) as `image`, exactly one driving video as `video` (the models_explore snapshot
  spells them `image_references` / `video_references`; check `models_explore` before the call). Upload local files
  with `media_upload` (type video for the reference).
- Price is not published: `est_credits` is null and the `get_cost` preflight sets it. The user OKs that number.
- Kling 3.0 Motion Control (std/pro tiers, background source switch) is a different lane and is not in the model
  registry yet; verify its current tool support when selected; a missing registry entry alone does not justify substituting Genjutsu.

Hand off every exact clip ID/file, shot identity, source draft/reference IDs, model/settings/seed support, provider job, measured duration and review state. Final promotion follows `studio`; selected clips go to `clip-assemble` or the selected editor with the measured audio-edit record.

## OpenRouter video (interface may change)
Read `studio` → `references/openrouter.md` for environment checks, live pricing/approval and credit caps. Discover via `/models?output_modalities=video` and `GET /videos/models`; check `supported_resolutions`, `supported_aspect_ratios`, `supported_durations`, `supported_frame_images`, `generate_audio`, `seed`, `pricing_skus` for the chosen model. SKU prices are cents, not USD. Video is not ZDR-eligible; explain a conflict with a required ZDR policy before submitting.

`POST /videos` requires `model` and `prompt`; duration is integer seconds. For image-to-video, `frame_images` uses `frame_type: first_frame` / `last_frame`, but the exact image-object entry shape is **unverified** here: check `supported_frame_images` and the current [video guide](https://openrouter.ai/docs/guides/overview/multimodal/video-generation) before building it from the selected frames. Do not invent the shape or omit a kept frame. Verify `input_references` separately; motion transfer is unconfirmed. Seed is not guaranteed. Set supported duration, resolution/aspect or pixel size, and `generate_audio` explicitly when selected (audio defaults true on audio-capable models).

This minimal text-to-video shape demonstrates submission/poll/download only. Use it only for an authorized text-to-video request; for image-to-video add the verified frame entries before submission. Run in the production folder with the selected `prompt.txt` and resolved model (example `google/veo-3.1`):

```sh
jq -n --rawfile prompt prompt.txt --arg model 'google/veo-3.1' \
  '{model:$model,prompt:$prompt}' > video-request.json
curl --fail --silent --show-error https://openrouter.ai/api/v1/videos \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" -H 'Content-Type: application/json' \
  --data-binary @video-request.json --output video-job.json
video_id=$(jq -er '.id' video-job.json)
while true; do
  curl --fail --silent --show-error "https://openrouter.ai/api/v1/videos/$video_id" \
    -H "Authorization: Bearer $OPENROUTER_API_KEY" | \
    jq -e '{status,generation_id,usage,error}' > video-status.json || break
  video_status=$(jq -er '.status' video-status.json) || break
  case "$video_status" in
    completed)
      curl --fail --silent --show-error "https://openrouter.ai/api/v1/videos/$video_id/content?index=0" \
        -H "Authorization: Bearer $OPENROUTER_API_KEY" --output video.mp4
      break ;;
    failed) jq '.error' video-status.json; break ;;
    pending|in_progress) sleep 30 ;;
    *) break ;;
  esac
done
```

Submission returns 202 with `id`, `polling_url`, `status: pending`; save the id before waiting. Poll about every 30 seconds. On interruption or failed polling, resume this id, never automatically resubmit. A failed job has `error`; explain it and resolve another paid attempt with the user. Register `video.mp4` only after successful content download. Completed status includes `generation_id`, `usage.cost`, unsigned URLs requiring auth and optional signed URLs with `expires_at`. Do not log/share signed URLs; the poll recipe keeps only status, generation id, usage and error. Record only job/generation ids, costs and durable file references. Outputs expire: immediately run `asset-persist` after download, then measure and review the clip as above.
