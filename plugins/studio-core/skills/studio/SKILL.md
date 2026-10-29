---
name: studio
description: "Creator Studio coordinator. Use for any request to make narration, voiceover, music, sound effects, images, video or podcasts; to set up Creator Studio or ask what it can do; to start, resume or revise a project or episode; to ask which models or defaults a project uses, or to change them. Handles first-run setup (projects folder, dependency check), seeds new projects with saved defaults, selects the stage skills, keeps the user's explicit choices, and saves resumable handoffs. A run ends at the finished file, its sidecar and status.md."
---

# Studio

The LLM coordinates existing skills directly. Each stage below points to a skill that says what it takes, what it makes and which tool it uses; follow those pointers. There is no engine, CLI or script to run: everything is instructions plus the tools you already have (connectors, FFmpeg, the browser).

## First run and setup
Do this when the projects folder is not configured, when the user asks to set up Creator Studio, or when they ask what it can do.
1. Read [capabilities.md](references/capabilities.md) and tell the user, in friendly words, what Creator Studio can do and how to start. Give a few example requests.
2. **Projects folder.** Resolve it in this order: a path in the request, `$CREATOR_STUDIO_VAULT`, then `vault_root` in `~/.creator-studio/config.json`. If none exists, ask where projects should live (suggest `~/CreatorStudio`), create the folder, and with the user's OK save `{"vault_root": "<absolute path>"}` to `~/.creator-studio/config.json`. If that file cannot be written, tell the user to set `CREATOR_STUDIO_VAULT` instead. Never infer a location from anywhere else, and never copy projects-folder contents into the repo.
3. **Environment check.** Run the checks in [dependencies.md](references/dependencies.md) for what the user will use, report ready / missing in one short list, and install what is missing as that file says. For OpenRouter, use only the exit status of `test -n "$OPENROUTER_API_KEY"` (never echo). If missing, tell the user to export `OPENROUTER_API_KEY` in their shell profile/environment, never in the repo, project file or chat; do not read or edit secret files. Public model discovery needs no key, so setup can continue while generation waits.
4. **First project.** Offer to create one with OpenRouter defaults selected from the live list (see "New project" and the [example seed](references/templates/project-defaults.openrouter.example.json)), then ask what they want to make. Providers are optional and mixable per modality: for example ElevenLabs direct for music/SFX, OpenRouter for TTS/images, Higgsfield for video. Select the model in `models.<modality>.default`, then use its provider’s `routes.<provider>.default` or exact per-model key; selecting OpenRouter does not redirect other providers.

## Resolve and resume
1. Resolve the projects folder as above. An explicitly supplied project or profile path can be read directly.
2. Resolve the project from the request, else the current production or `$CREATOR_STUDIO_PROJECT`; ask only if ambiguous. A project that does not exist is created (see "New project"). Read the project folder ([project-file.md](references/project-file.md)) and the current production status. On project switching, reload that project's defaults and IDs; do not carry another project's voice, timing or assets across.
3. Reuse a suitable existing episode/production record. Otherwise keep `productions/<production-id>/status.md` within that project. Preserve existing IDs; assign a new production label only for new work. Read [production-handoff.md](references/production-handoff.md) when creating/updating the record or resuming an incomplete handoff.
4. Load only the current stage skill and its selected references. Read only the files needed to identify inputs and selected outputs.

## New project
Create from [templates/project.json](references/templates/project.json), merged with `<projects>/project-defaults.json` when that file exists (see "Seeding a new project" in project-file.md); no existing project, config or defaults file is required. Ask once, in one message, only for what the request left open: name, kind, model choices for the selected modalities, and the voice. For OpenRouter, follow [live discovery](references/openrouter.md#discovery-and-paid-step-preflight), not a fixed model shortlist; for other providers use [the model references](references/models) and live provider listings. Offer a short list with one line on why each fits and its estimated credits (USD for OpenRouter; unknown amounts labeled).

For OpenRouter, save the user’s exact live choices to `models.voice.default`, `models.transcription.default`, `models.image.default` and `models.video.default` as applicable; "decide each time" leaves the corresponding default `null`. For voice, resolve and save a `voices[]` profile with that exact model id and a supported voice before rendering; `models.voice.default` alone is not a render profile. Set `routes.openrouter.default` to `"openrouter"` when selected, and offer `allowed` / `locked` only if wanted. The [example seed](references/templates/project-defaults.openrouter.example.json) shows the shape, not model recommendations: replace its fictional ids with the user’s choices or `null`. Offer to save reusable choices in `<projects>/project-defaults.json` through the same merge mechanism. Save the project to `<projects>/projects/<slug>/project.json`, create `productions/`, and tell the user in one line where it is and that defaults can be changed any time.

## Project defaults
The project file holds the saved defaults a run follows: `models.*`, `defaults.*`, `routes`, `tools`, `spend`, `audio`. Precedence, the meaning of `locked`, and how to save a default are in [project-file.md](references/project-file.md). In short:
- Use the project's default when the request names nothing, and record "from project default" in `status.md`.
- If a default cannot do what the request needs (for example a Soul model asked to carry an outfit Element), explain the specific conflict and resolve it with the user. Never swap silently.
- Save a default to `project.json` only when asked ("make X the default", "always use X here") or during setup. "Just this episode" goes in `status.md`. If the scope is unclear, treat it as this production and offer once to save it as the default.

## Choices and authorization
- The user's latest explicit decisions override earlier production choices and project defaults. Preserve project identity, creative rules and actual review/spending constraints. If a choice conflicts with a locked rule or supported route, explain the conflict and resolve it with the user; never silently substitute a model, platform, voice/character ID, seed, quality, route or reference image.
- Record choices and their scope (whole production, scene or take), including draft/final settings. Get IDs from the profile, asset records or provider listings; never invent them. Ambiguous references such as "that image" require resolving the exact asset rather than assuming newest means selected.
- Follow the user's explicit choices and any spend limit in the project file; ask before any paid step not already authorized. Keep the project's user gates, test-first listening, CapCut manual work and explicit Descript-call rules. Never record an approval on the user's behalf.
- Before paid generation, use the actual route's non-billable cost check when available; track cumulative spend against the project's spend limit. Unknown cost is not zero. If authorization does not cover the action/cost, ask for what is missing. Poll slow jobs; avoid duplicate submissions.
- For `openrouter`, read [openrouter.md](references/openrouter.md): check only that `OPENROUTER_API_KEY` is set, discover live models/prices, and apply `spend.credits_per_usd` before comparing USD costs with credit caps. Missing conversion or unknown cost requires resolving authorization with the user. Never ask for, print or store the key value.
- Use installed skills/connectors/CLIs, curl for OpenRouter, or the available browser tools. Claude-specific tool names and the `higgsfield` prompting skill are environment-specific references, not guarantees. **If a selected dependency is absent, check and install it as [dependencies.md](references/dependencies.md) says**, ask the user only for what only they can do (sign-ins, connectors, approvals), and save the next action. Do not build a replacement integration or silently change route.

## Navigate stages
Choose only stages needed for the request; a revision can start at its affected stage.

| Stage | Skill | Takes | Makes | Tool |
|---|---|---|---|---|
| Script | `narration-and-story-scripting` (`two-host-podcast-director` for a two-host show) | topic, source or the user's script; project rules | narration script (text + JSON beats), estimated length | Claude |
| Emotional direction | `emotional-director` | approved script, project's locked delivery foundation | the same script plus a model-agnostic emotion layer (per-passage emotion, intensity 1-5, energy, pace, arc); words verbatim | Claude |
| Voice direction | `elevenlabs-voice-director` (`elevenlabs-voice-director-sleep` for sleep / meditation / wind-down narration), `ai-voiceover` | approved script (with the emotional direction layer when that stage ran), project voice | voice pick, settings, ready-to-render text, split map, pause notes | Claude |
| Narration render | `speech-render` | render text, voice, model, settings | speech files in narration order | ElevenLabs (connector or web); OpenRouter (curl) |
| Narration edit (pass 1) | `audio-post-mix` → "Narration edit first" | ordered speech files | the joined narration and its edit/timing table | FFmpeg |
| Pace and pause | `pace-and-pause` | joined narration, directed script, pause notes (`elevenlabs-voice-director` → "Pause notes"), `audio.pace` | the hand-placed pauses and rests (inserts filled with room tone), edit list, insert table | whisper.cpp, FFmpeg, Rubber Band |
| Voice lock | `audio-treatment` → "Voice lock" | paced narration, the project's voice treatment | the locked voice (one hashed file) and the timing table measured from it | FFmpeg |
| Visual direction | `visual-director` | locked narration + measured timing, script (with the emotional direction layer when present), pause notes, project visual rules, existing-asset inventory | `visual-plan.md` + `timeline.json`: shot plan, treatment and cost per shot, shot briefs for a separate prompt-writer, music/SFX spotting map, budget tiers, rule checks (plan only) | Claude; ffprobe, ffmpeg, jq |
| Stills | `image-generate`, then `image-verify` | beats or shot list, character references | still images + QC report | Higgsfield (prompts via the `higgsfield` skill); OpenRouter (curl) |
| Clips | `video-from-image` | selected stills or a motion reference | video clips | Higgsfield; OpenRouter (curl; verify frame support) |
| Music and effects | `ai-music-and-sound` (brief), then `sound-render` or `suno` | script beats or the spotting map's cue list, brand sound | music and effect files tied to cues (no effects is valid) | ElevenLabs; Suno in the browser |
| Cue and level plan (pass 2) | `audio-post-mix` | the locked voice's timing table, the spotting map, music/effect files, project levels | cue and level plan (applied as track automation in the editor), preview-only mixes, level QC | FFmpeg (QC) or the selected editor |
| Master | `audio-master` | audio-only program, or the editor's export, and the project loudness target | mastered audio (audio-only work) or the export's loudness report | FFmpeg |
| Treatment (optional) | `audio-treatment` (after Master by default for audio-only work; can also run on fragments before the narration edit; for an episode with picture, voice treatments run before the voice lock: `audio-treatment` → "Voice lock") | mixed/mastered audio or fragments, named treatment from the project | treated audio + loudness report | FFmpeg |
| Conform | `episode-conform` (spec; the agent builds the conform toolkit in the project's `tools/` folder on first use) | edit plan (`timeline.json`), cue and level plan, locked voice and timing table, breather list, selected media, previous version | timing tables, the editor project built in one pass, QC evidence, producer fix list, a new version folder | the selected editor (FilmCraft) headless; FFmpeg for measurement |
| Assembly and QC | `filmcraft` (or the selected editor); `clip-assemble` for previews and single-layer files; `captions-and-clipping`, `capcut`, `descript`, `opus-clip` when needed | stills/clips in shot order, the locked voice, music cues and SFX each on its own track, timing table | final video (or shorts), checked | FilmCraft, FFmpeg, CapCut (manual), Descript |
| Finish | `studio` (this skill) | final file | sidecar `<file>.json` + `status.md` line | `shasum -a 256` / `sha256sum` |

**Episode order (with picture).** The voice clock is written once and everything else is timed against it, so the stages run in this order:
1. Render (`speech-render`), then the narration edit's join (`audio-post-mix`, pass 1).
2. Hand-placed pauses and pacing (`pace-and-pause`), starting from the pause notes. Pauses are placed by hand after generation, never as tags.
3. Voice lock, with the timing table measured from the locked file (`audio-treatment` → "Voice lock").
4. Visual direction and spotting (`visual-director`), on the locked clock.
5. Cue and level plan (`audio-post-mix`, pass 2), from the spotting map.
6. Conform (`episode-conform`), then assembly QC and the user's gate.

Any change to the voice after step 3 (an inserted pause, a re-rendered line) is a new lock version, and every later table is rebuilt from it: `audio-treatment` → "Voice lock".

An episode is conformed from timing tables in one pass and rebuilt, never hand-patched: `episode-conform`. Section breathers (music-only breaks between story sections) are an assembly step on the voice, music and SFX tracks: `filmcraft` → "Section breathers".

Every layer of an episode (voice, music cues, SFX, intro/outro brand pieces, picture) goes on its own track in the editor, and FFmpeg only preps the voice and measures; a pre-mixed file is a preview, never the episode audio: `filmcraft` → "Assembly split". The signature intro and outro: `filmcraft` → "Signature intro and outro".

Heavy local jobs (FFmpeg, speech-to-text, exports) follow `references/local-jobs.md`: map only the streams you use, seek before `-i`, one job at a time under the memory and disk watchdog, and scratch outside cloud-synced folders. A check tool built from a spec is proven on a planted defect and a clean input before it is trusted (same file, "Prove a check before trusting it").

Visual concepts/generation can happen alongside audio. Final shots, captions and effects use actual narration-edit timings, not estimated script length. Review voice, music, effects and visuals together; a loudness pass alone is not a creative review.

## Drafts, targeted revisions and final production
Record each draft/take before review. Select exact outputs; do not treat generated as approved. "Approve scenes two and five; redo scene three" keeps two and five selected and changes only three plus affected assembly/timing/review work identified from the handoffs.

For "make high-resolution versions of approved drafts", establish whether the requested operation enhances/upscales the existing asset or regenerates it. Preserve the chosen source and settings, record supported seed control (same seed does not guarantee identical content), and explain unsupported routes before proceeding. Do not promote rejected or unselected takes. Keep replaced outputs and lineage; mark them superseded only when their replacement is selected.

## Save and hand off
Save provider outputs promptly into the production folder (`asset-persist`; expiring URLs are why). Cleanup never deletes a source asset: `asset-persist` → "Cleanup: keepsake rule". Keep hashes and the inputs each output came from. The coordinator updates shared production status after each meaningful decision/output and before leaving a stage or pausing. Optional delegated work gets a bounded assignment, exact inputs/settings and expected saved-output handoff; workers do not overwrite shared status.

The run ends with the finished file, its sidecar `<file>.json` ([output-sidecar.md](references/output-sidecar.md), including the music and SFX `credits` a publishing step must carry into the description) and `status.md` updated with the file location, review state, outstanding defects, next action and actual provider costs (unknown amounts labeled). Publishing is optional and separate: hand the file + sidecar to a separate publishing tool if wanted. Never commit media, projects-folder contents or secrets to the repo.

## Model reference
[references/models/](references/models) holds dated snapshots of model ids, capabilities, routes and estimated credits for Higgsfield, ElevenLabs, Suno and OpenRouter (TTS, STT, image, video). They are lookups, not proof of current limits or prices: confirm with the connector's live schema or a cost check before spending.

For ElevenLabs speech on either route, resolve the model from `voices[].model` (explicit production choices become scoped profile overrides), never a code/provider default. `models.voice` guides setup and constrains choices; it does not replace a cast profile. v4 is recommended for new work; selected v3 stays valid. Read [elevenlabs-tagging.md](references/elevenlabs-tagging.md) for tag support, limits, continuity, route controls and model-change listening review (shared v3/v4 syntax, model-specific delivery).
