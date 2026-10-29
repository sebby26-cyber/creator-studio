# Production status and handoff

Use an existing suitable production/episode record; otherwise adapt this small Markdown record at `<project>/productions/<production-id>/status.md`. This is a content checklist, not a schema or state machine. Link to profiles and manifests rather than duplicating them. The coordinator owns updates.

```markdown
# <existing production/episode identity or new label>
Project: <slug and exact profile path>
Request: <current scope>
Stage / review state: <where work stands>
Explicit choices: <tool/model/route, draft and final quality, seeds, voice/character/reference IDs; scope of each choice; mark values taken from the project's defaults as "from project default">
Selected outputs: <scene/beat/segment -> exact artifact ID/file, draft/take/version and generated/selected/approved/rejected/superseded state>
Timing/edit: <record path and source narration-edit ID/version; provisional or measured>
Outstanding fixes: <affected pieces, defects and required reviews>
Authorization / spend: <existing user decisions/gates/caps, actual costs and unknown amounts; no invented approvals>
Next action: <bounded action, exact inputs and any missing dependency/input>
```

Each output handoff identifies project + production, scene/beat/segment or dialogue line/chunk, exact file/artifact reference, draft/take/version and review state. Include model, voice/character/reference IDs, seed (or unsupported), relevant settings, provider job/node/result reference when available, defects and next action. Add intended order and measured duration/placement when the next stage needs timing. Do not infer identity/order from filenames or download completion order.

For speech, record each take’s actual render model here and in its [existing sidecar](output-sidecar.md), rather than relying on the current profile. On a v3/v4 switch or mixed-model handoff, compare the recorded models and flag affected tagged takes `pending_review` in **Selected outputs**, with re-listening in **Outstanding fixes** / **Next action**; preserve prior review history. Apply the [model-change listening review](elevenlabs-tagging.md#model-changes-and-listening-review) before final approval.

For OpenRouter, record route `openrouter`, exact model id, supported voice, job/generation id, estimated/actual USD cost and any explicit credit conversion used for caps. Unknown costs stay unknown. Never record authorization headers, key values or signed URLs; retain durable files and job ids instead.

For a narration edit, a simple table is enough: segment/line/chunk, selected asset, source in/out, measured program start/end, native leading/trailing quiet and any inserted pause. Preserve breaths and project timing policy. Bind captions/shots/sound cues to this edit version. A changed edit marks affected timing and assembled outputs for recheck; unrelated approved assets stay selected.

For music/SFX, record cue ID -> provider job/node/result (or library source) -> durable asset ID/file, intended offset/duration, selected take, applied fade/gain/ducking settings, and its licence and exact credit line from the music shortlist. The final file's sidecar carries those credit lines forward (`credits` in [output-sidecar.md](output-sidecar.md)). A provider batch result must be matched by its reference; if the provider supplies no binding, generate separately or resolve explicitly before mixing.

Final promotion records the selected source draft, enhancement/upscale versus regeneration, requested/applied model/settings and seed support, final output, review results and supersession. Keep old IDs and lineage. Write the final file's sidecar `<file>.json` next to it ([output-sidecar.md](output-sidecar.md)) and add its line to `status.md`.
