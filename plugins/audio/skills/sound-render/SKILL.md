---
name: sound-render
description: "Render music cues, whole tracks and SFX with ElevenLabs (Music / Sound Effects) by route (mcp | web) from a music brief or the cue list's effects. Uses the selected route, checks cost, and binds every output to its cue and durable asset."
---

# Sound render (ElevenLabs Music + SFX)

OpenRouter does not expose ElevenLabs music, SFX or voice-library access. Speech tags such as `[applause]` are unreliable substitutes for separately timed effects (the `studio` skill → `references/elevenlabs-tagging.md`). These stay on the direct ElevenLabs `mcp` / `web` routes below; never redirect them because OpenRouter is selected for speech.

Input: a music brief (usually from `ai-music-and-sound`, `source: elevenlabs`) or the cue list (effects per cue,
from `audio-post-mix`). Output: music files (one per cue, or one whole track) and effect files (one per listed
effect, labelled with its cue and offset). Pairs with `speech-render` (the voice); `audio-post-mix` mixes all three.

## Rules
- **Models from explicit production choices, else project/config lookup; never invented:** `studio` skill → `references/models/elevenlabs.json` `sound`: music
  `eleven_music_v1 | eleven_music_v2 | eleven_music_v2_5` (default v2.5), SFX `eleven_text_to_sound_v2`
  (live connector schema, 2026-10-08). The project's `models.music.default` / `models.sfx.default` apply when the request names none; an unknown id is
  refused, never swapped.
- **Route** from the explicit production choice, else the project (`routes.elevenlabs.default`):
  - `mcp`: `creative_create_flow` -> `creative_add_flow_node` per cue/effect (`node_type` music or sfx,
    `model_parameters` per the schema: music `duration_seconds` 3-600, `lyrics_type`, `instrumental`; SFX
    `duration_seconds` 0.5-30, `prompt_influence`, `loop`) -> **preflight** `creative_run_flow_nodes` with
    `estimate_only: true` -> after the OK, the same call with `generations_count: 1` (tool default is 4 = 4x cost)
    -> poll `creative_get_flow_run_status`. Outputs are flow URLs that expire in about 2 h: persist at once.
  - `web`: elevenlabs.io (Music / Sound Effects) in the user's signed-in browser, driven by Claude (Claude in
    Chrome, or a headless browser); paste prompts; read the cost before Generate; download; persist with
    the brief id. Requires available signed-in browser tools; report a missing connection instead of promising browser access.
- **Prices:** SFX 40 credits per second when the duration is set (ElevenLabs docs; always set the duration).
  Music has no published price: treat it as unknown until the `estimate_only` preflight gives the price. Never
  treat an unknown as zero.
- **Prompting** (from `ai-music-and-sound` and `audio-post-mix`): music cues carry the brief's feel (genre,
  mood, tempo, energy at hook / build / payoff), reference a vibe never a copyrighted song, and stay instrumental
  under narration (vocals fight the voiceover); continuous beds, evolving music or separate cues according to direction, including continuity across narration boundaries. SFX are the accents
  the plan lists (whoosh on a hard cut, riser into the payoff, stinger on the logo, subtle ambience), one effect
  per node. Read `creative_get_model_guide` for the model before writing a new kind of prompt.
- Follow existing authorization, spend caps and review gates from `studio` and the production record. Do not resend a call that looks slow; check its status.

## In Creator Studio
- **Takes:** the music brief or cue list, model, route and settings.
- **Makes:** music and effect files, each tied to its cue (an empty effects list is valid: skip effect calls).
- **Tool:** ElevenLabs Music / Sound Effects through the connector or the web app. Confirm current model IDs and parameters with the tool; config is a lookup snapshot.
- **Save:** put the files in the production folder and add a line to `status.md`.

Maintain explicit `cue ID -> provider job/node/result ID -> durable asset ID/file` bindings; also record model,
prompt/settings, take/version, review state, intended offset and measured duration. Do not match batch outputs by
download order. If the provider lacks binding metadata, render separately or resolve each result before mixing.

Run the `ai-music-and-sound` licence check on kept music. Hand selected outputs and bindings to `audio-post-mix`,
which places them against the finalized narration edit. Record defects and next action in production status.

## Output fields
- Music: one file per cue in the brief (a brief without cues gives one), or one whole track. Record `uri`,
  `duration_s` and the provider job ID (flow/session id).
- Effects: one file per listed effect: `{uri, label, cue, offset_s, duration_s, model, prompt}`.
- Source: new in Creator Studio (2026-10-08). The renderer for the ElevenLabs Music/SFX source that
  `ai-music-and-sound` recommends; tool and parameter names from the live claude.ai ElevenLabs connector.
