# Project file

A project is a folder of your own settings and files. It lives outside this repo, in your **projects folder** (the one you chose at first run). The repo and plugins hold no project data.

```
<projects>/
  project-defaults.json        optional: your personal defaults, merged over the template for every NEW project
  projects/<slug>/
    project.json               project settings (this document)
    workflow.md                optional: the project's own rules and review checkpoints
    brand/                     optional: brand files
    tools/                     optional: helper tools the agent builds for this project on first use (watchdog, conform toolkit)
    productions/<production-id>/
      status.md                resumable handoff (see production-handoff.md)
      …                        the files made in that production, and their sidecars
```

## Finding the projects folder
In this order: a path given in the request, `$CREATOR_STUDIO_VAULT`, then `vault_root` in `~/.creator-studio/config.json`. If none is set, this is a first run: the `studio` skill asks where projects should live and offers to save the answer to `~/.creator-studio/config.json`. Never guess a location and never copy projects-folder contents into this repo.

## Finding the project
From the request, else the current production, else `$CREATOR_STUDIO_PROJECT`; ask only if it is ambiguous. A project that does not exist yet is created from the template (see "Seeding a new project").

## Keys
No key is required. A missing key means "no default": the skill uses its own guidance or asks. `null` means the same. Unknown keys are kept and ignored, so you can add your own.

| Key | Meaning |
|---|---|
| `slug`, `name`, `kind` | Identity. `kind` is plain words (`youtube`, `podcast`, `shorts`, …). |
| `routes.<provider>.default` | How a provider is reached: `mcp` (connector), `cli`, `web` (signed-in browser), or `openrouter` (curl against the OpenRouter API). Per-model overrides go in `routes.<provider>.<model>`. |
| `models.image`, `models.video`, `models.motion`, `models.voice`, `models.transcription`, `models.music`, `models.sfx` | The model to use for that job. `default` is used when the request names none. `allowed` (image, video, voice, transcription): when non-empty, only recommend from this list. `locked: true` makes the default a project rule: change it only with the user's explicit OK in the current request. For voice, resolve the selection into `voices[].model` before rendering; this default never replaces a cast profile silently. |
| `defaults.mode` | `economy`, `balanced` or `quality`: how to weigh cost against quality when recommending a model. |
| `defaults.aspect`, `defaults.image.*`, `defaults.video.*` | Aspect ratio, image count and quality, video duration and resolution. Per-media values win over `defaults.aspect`. |
| `defaults.seed` | Seed to use where the model supports one. A same seed does not guarantee identical content. |
| `defaults.captions`, `defaults.draft_then_final` | Burn in captions by default; render cheap drafts before finals. |
| `tools.prefer`, `tools.avoid` | Skills or tools to favor or leave out (for example `{"prefer": ["filmcraft"], "avoid": ["capcut"]}`). A request that names a tool overrides these. |
| `spend.limit_credits`, `spend.ask_above_credits` | Project spend cap, and a per-step price above which to ask even when spend is otherwise authorized. |
| `spend.credits_per_usd` | Optional positive, user-set conversion for USD routes: credits = USD × this rate. No implicit exchange rate; missing or null means no default. Ask for a conversion before evaluating a credit cap/threshold against USD, or for an explicit USD authorization that resolves the cap. Keep USD and converted credits in the spend record; never add unlike units. |
| `voices`, `characters`, `wardrobe` | Cast voices (label, provider, voice_id, model, settings), character references (Soul id, Elements) and outfit assets. The TTS model ALWAYS comes from the resolved `voices[].model`, never a code/provider default. Resolve explicit production choices into the profile for their scope. IDs come from the provider or earlier outputs; never invent them. |
| `gates` | User review checkpoints (for example script approval, listening test). Never record an approval on the user's behalf. |
| `audio.loudness`, `audio.export` | Loudness target (`integrated_lufs`, `true_peak_dbtp`, optional `lra`) and export settings (`sample_rate`, `channels`, `bitrate`). |
| `audio.mix_levels_db`, `audio.treatments`, `audio.chunking`, `audio.max_chunk_chars`, `audio.closing_fade_s` | Starting mix levels, named tone treatments, narration chunk controls and the closing fade. `audio.chunking.target_chars` is the preferred chunk target; `audio.max_chunk_chars` caps final outgoing text including tags. Both must stay within the selected route/endpoint ceiling; missing/null sets no project target. Split at paragraph/sentence breaks, never within a tag/sentence. Conservative model targets and continuity rules: [elevenlabs-tagging.md](elevenlabs-tagging.md). `audio.chunking.split_at` (opt-in): `"section"` (missing/null means this: chunk by size, unchanged), `"sentence"` (a split point and its own render request at every sentence end) or `"phrase"` (also at every comma, colon or dash the director marks as a hold, the one case that splits inside a sentence). Pause lengths at split points still come from `audio.pace`. Rule: `elevenlabs-voice-director` → Step 6 "Split granularity"; rendering and the A/B listening test before first use: `speech-render` → "Split-point rendering". The sleep register does not read it. |
| `audio.pace` | Settings for `pace-and-pause` (its "Project settings" table has every default). **Opt-in:** missing or `null` = pass-through; `ramp` missing or `false` = neutral mode; `ramp: true` = ramp mode. Keys: `ramp`, `base_wpm` (required), `rate_definition`, `mark_ratio {slower, steady, brisker}` (required), `tolerance_pct`, `max_stretch` (required in neutral mode; never above 1.10 in ramp mode), `wpm_ceiling {start_wpm, end_wpm}`, `rest_ramp {sentence, paragraph, transition: {start_s, end_s}}` (required in ramp mode), `rest_ramp_curve`, `rest_max_s`, `tail_min_s`, `land_rest_s`, `rest_follows_taper`, `rest_tolerance_pct`, `rest_gate_pct`, `crossfade_ms`, `silence_detect {noise_db, min_s}`, `room_tone_source` (`from_program`, a file, or `none`), `taper`, `end_tail_s`, `long_sentence_words`, `punct_pause_s {comma, colon, dash}`, `punct_place`, `sentence_rest_s {min, max, after_fast_min}`. |
| `voice.sleep.hold_tag` | The one hold tag for sleep / wind-down narration (`elevenlabs-voice-director-sleep`), placed at every paragraph and chunk start and at re-tags. Skill default `[quietly]`. `voice` is a settings object, separate from the `voices` cast list and `models.voice`. |
| `voice.sleep.endings_tags`, `voice.sleep.banned_tags`, `voice.sleep.untested_tags` | Tag lists for the sleep register: tags allowed only in the closing lines (default none), tags never used (default: performance / arousal tags), and tags usable only in an authorized A/B (default `[calm]`, `[peaceful]`, `[hushed]`, `[slowly]`, `[gentle]`). A project may move a tag between lists. |
| `voice.sleep.retag_every_sentences`, `voice.sleep.retag_every_chars` | Re-tag inside a long paragraph after N sentences or N characters since the last tag, whichever comes first. Defaults 2 / 250. Here `null` turns re-tagging off, so the template carries the defaults. |
| `voice.sleep.split_min_rest_s`, `voice.sleep.chunk_chars` | A designed rest at or above this many seconds becomes a chunk split plus a split-map entry (default 1.5); chunk size band `{min, target_min, target_max}` counting tags (default `{800, 1500, 3500}`), still bounded by `audio.chunking.target_chars`, `audio.max_chunk_chars` and the route ceiling. |
| `voice.sleep.stability`, `voice.sleep.carry_over_chars` | Stability for the main render (default 0.5) and characters of `previous_text` / `next_text` sent each way (default 500). |
| `voice.sleep.post_silence` | `allowed`, `refused` or `ask` (default): may assembly insert silence for long rests? Record the user's answer here, never in a skill. |
| `voice.sleep.rest_fill`, `voice.sleep.gain_match`, `voice.sleep.em_dash` | Fill for inserted rests `{source: room_tone | from_program | none, crossfade_ms}` (default `room_tone`, 50); pre-join level matching `chunk_median_lufs` (default, set in the template) or `null` (off); em dashes `allowed` (default) or `banned`. |
| `voice.sleep.base_wpm`, `voice.sleep.qc` | Measured narration rate for length estimates (default `null`: measure on a test render); pause QC thresholds `{noise_db, min_s, gap_flag_ratio, class_band_pct, join_tolerance_pct}` (default `{-40, 0.3, 2, 20, 20}`). |
| `voice.sleep.blur_watchlist` | Word sequences known to blur on this project's voice: strings or `{sequence, added, source, fix}` objects. Missing or `null` = empty. The sleep director fixes each hit before render and adds new ones after the transcript audit. |
| `visual.canvas`, `visual.text`, `visual.look`, `visual.motion`, `visual.hold`, `visual.reuse`, `visual.prompting`, `visual.tiers` | Visual rules for `visual-director`, as data instead of prose: canvas `{width, height, fps}`; on-screen text `{body: none | allowed, allowed_windows, note}`; look `{palette, temperature, guards, grade, reference_frames}`; motion `{allowed_moves, kb_scale_max, banned, min_clip_speed}`; holds `{min_s, max_s, split_long_stills, cut_after_last_word_s, cut_before_first_word_s}`; reuse `{prefer_sources, exclude}`; prompting `{method, agent}`; budget tiers `{default, attempts_margin}`. Each missing key uses the skill's default or asks. |
| `qc.checklists` | The project's image or video QC checklists, by name. |
| `notes` | Anything else worth remembering. |

Project values (names, voices, checklists, gates) come from this file, never from a generic skill. ElevenLabs recommends v4 for new work; v3 remains valid when selected by a project/voice. Neither is a code default. Tags share syntax; delivery is model-specific ([maintainer-observed]). Follow [model-change listening review](elevenlabs-tagging.md#model-changes-and-listening-review) on switches or mixed-model work, including the user’s listening gate again for an approved production. Record each take’s actual render model in its [sidecar](output-sidecar.md) and in [status.md](production-handoff.md), so a profile change flags affected tagged takes for re-listening.

Providers are optional and mixable per modality. Choose each modality’s model in `models.*.default`, then reach that model’s provider through `routes.<provider>.default` or its per-model key. For example, keep ElevenLabs direct for music/SFX, OpenRouter for voice/images and Higgsfield for video; an OpenRouter default does not reroute every modality.

## OpenRouter route
`routes.openrouter` uses the same fields as other providers: `default` is `"openrouter"` when selected; an exact model-id key overrides it (for example `"openai/gpt-image-2": "openrouter"`). Missing or null means no default. The route is curl against `https://openrouter.ai/api/v1`, with `Authorization: Bearer $OPENROUTER_API_KEY` read only from the shell environment. There are no credential or base-URL fields in the project. The user exports `OPENROUTER_API_KEY`; the agent never asks for, prints or stores its value.

Store the exact OpenRouter id in `models.voice`, `models.transcription`, `models.image` or `models.video` (`default` / `allowed` / `locked` as above); the upstream author prefix is part of the id, not the selected route. A voice profile on this route uses `provider: "openrouter"`, its exact model id and a voice from that model’s `supported_voices`. The profile's `model` is the rendering source of truth; `models.voice.default` guides setup and `allowed`/`locked` constrain selection, never silently replace a cast model. Resolve missing profiles before rendering. Explicit production choices remain in `status.md` as scoped profile overrides. These choices follow the same precedence and unsupported-capability rules as existing routes. Never translate a direct-provider model or voice id silently. See [openrouter.md](openrouter.md) for live discovery, price checks and transcription; [models/openrouter.json](models/openrouter.json) is the offline/unreachable-list fallback.

## Order of precedence
1. What the user says in the current request, for the scope they name (the whole production, one scene or one take).
2. The production's `status.md` **Explicit choices**.
3. The project file: `locked` models and `gates` are rules (a conflict is explained and resolved with the user, never swapped silently); `models`, `defaults`, `tools` are defaults.
4. Skill guidance and the model reference files in `models/`: recommend one, say why, and record it.

When a project default is used, write it into the production's `status.md` as "from project default" so the production stays reproducible if the default changes later.

## Saving defaults
Edit `project.json` only when the user asks for a default to be saved ("make Kling the default video model", "always 9:16 for this project") or during project setup. Change only the keys involved, keep everything else, and say in one line what changed. A choice for "this episode only" goes in `status.md`, not here. When the scope of a choice is unclear, treat it as this production and offer once to make it the project default.

## Seeding a new project
Every new project starts from [`templates/project.json`](templates/project.json). If `<projects>/project-defaults.json` exists, its keys are merged over the template first (objects merge key by key, lists are replaced; scalars and `null` replace the template value), so a personal default such as a preferred voice or loudness target seeds every new project. With no defaults file, use the template alone; no existing project or config is required once the projects folder is chosen. Then ask only for what the request left open (kind, models for the selected modalities, voice), fill `slug` and `name`, and save to `<projects>/projects/<slug>/project.json`.

For OpenRouter, [`templates/project-defaults.openrouter.example.json`](templates/project-defaults.openrouter.example.json) is a fictional starting shape for the same merge, not a separate setup path. Replace every example model id with the user’s choice from [live discovery](openrouter.md#discovery-and-paid-step-preflight), or `null` for "decide each time", before saving it as `<projects>/project-defaults.json` if the user wants reusable defaults. It sets `routes.openrouter.default` and the voice, transcription, image and video model defaults; `allowed` and `locked` are optional. Keep other providers’ choices for modalities they will handle. Never put credentials in either file.

## Adding keys later
Add the key to the template, add a row to the table above, and make the skill that uses it read it. Existing projects keep working: a missing key is "no default", and the agent offers to add it when the user asks. Files from earlier versions (`project_version` 2 or lacking `models`) stay valid.
