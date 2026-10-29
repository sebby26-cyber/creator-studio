---
name: image-generate
description: "Generate stills through OpenRouter or character-consistent stills on Higgsfield (Soul id, character/outfit Elements, multi-reference composites and edits, outfit-only assets, scenes). The coordinator preserves selected model, route, seed, references and draft/final settings; Higgsfield prompts are written with the `higgsfield` prompting skill."
---

# Image generate

- **Takes:** beats or a shot list, selected character/outfit references, and the model, route, seed and quality choices.
- **Makes:** still images (one per shot or request), plus outfit or composite stills when asked.
- **Tool:** Higgsfield (connector, CLI or web) or OpenRouter (curl); Higgsfield prompts written with the `higgsfield` skill.
- **Save:** put the files in the production folder and add a line to `status.md`.

## 1. Resolve inputs
`studio` supplies project/production identity, selected character/reference assets and explicit model/route/seed/quality choices. The model is the user's named model for this scope, else the production's recorded choice, else the project's `models.image.default` (respecting `allowed` and `locked`), else recommend one from the selected provider’s model reference (`studio` → `references/models/higgsfield.json` or `references/models/openrouter.json`) with the reason and record it. Resolve other defaults (`defaults.image.*`, `defaults.seed`) from the project and technical parameters from the specialist guidance/current tool schema.
- For OpenRouter, use the section below (`routes.openrouter`, route `openrouter`); the Soul/Element and Higgsfield steps below apply only to Higgsfield. Preserve the exact selected reference files; provider-specific ids are not interchangeable.
- Identity: a Soul id rides on `soul_2` / `soul_cinematic` only (one Soul per image). Other image models carry a
  character through its Element (`<<<uuid>>>` in the prompt). Outfits/scenes are Elements (category prop/
  environment) or plain text.
- Route per project (`routes.higgsfield`): `mcp` (mcp__higgsfield__generate_image), `cli` (`higgsfield generate
  create <cli id>`; ids differ: soul_2 -> text2image_soul_v2, nano_banana_2 -> nano_banana_flash; CLI Soul param
  `custom_reference_id`), `web` (settings checklist). Price can depend on route and on the user's Higgsfield plan (Seedream 5 Lite was 1 credit via MCP/CLI but can be free on the web on some plans); check live before spending.
- If the selected model cannot use the chosen identity/references, explain that limitation; preserve the choice until the user resolves it.

## 2. Write the Higgsfield prompt (mandatory method)
Creator Studio writes every Higgsfield prompt with the **`higgsfield` skill**
(third-party, referenced not vendored): load its root `SKILL.md`,
`skills/higgsfield-prompt/SKILL.md` and the model sub-skill its routing table names; take presets and parameters from
its `specs/model-specs.yaml`; check the aspect ratio against the model. Respect the project's prompt rules in its
`workflow.md` (for example, a rule that prompts describe scene, framing and lighting only; see the project file). Use the selected model sub-skill and verify installed dependency availability.

## 3. Generate on Higgsfield under existing authorization
1. Preflight: same call with `get_cost: true`, `use_unlim: false`; read **`credits_exact`** (not `credits`).
2. Check existing authorization and user-set caps/gates; record the exact price before generating.
3. Submit exactly once (`count: 1`; `use_unlim: true` only when the user explicitly asks: it spends the free-trial
   allowance). Wait with `jobs_wait`.
4. Persist at once with `asset-persist`, then run the project's QC (`image-verify`) and gates.

## 4. Outfit assets
Takes an outfit description; makes an outfit reference. "make an outfit asset black top and denim" means one still of the garments only
(no person, plain background; use the selected compatible text-to-image model, else the project default), then:
1. save the still (`asset-persist`);
2. make a Higgsfield Element from it with `mcp__higgsfield__manage_reference_elements` (read the tool's schema first;
   same category as the project's existing outfit Element; get the user's OK if its schema shows a cost);
3. record the outfit in `status.md`: short name, garments, the still's file and the Element ID.
A recorded outfit is then usable by name like a project outfit ("<character> wearing <outfit> ..."): Element models
put `<<<element id>>>` in the prompt; Soul models cannot carry an outfit Element; using a text description instead requires an explicit resolved choice.

## 5. Composites and edits
Takes a character reference, plus optional stills (a pose, a layout) and/or an outfit reference (+ scene text);
makes one new composite still. Reached by plain English: "combine", "put her in", "use that
dress/outfit/look", "swap her outfit", "edit that image". Requires a model supporting `image.multi_ref` (plus
`image.edit` when the request says edit), so select a compatible reference-image model under the recorded choices
(e.g. `nano_banana_2`); Soul models cannot, so explain any conflict with a locked Soul runbook.
- A request in parts runs in its stated order and the combining part takes what the earlier parts made: "make an outfit
  asset ..., then a standing pose of the character, then combine them into the character sitting in a coffee shop"
  means an outfit still, a pose still (no outfit), then one composite (character Element + pose still + outfit
  Element, scene "sitting in a coffee shop").
- Reference stills go in `medias` (a prior Higgsfield job id is passed as is; a local file is uploaded with
  `media_upload` first). The reference-image role name differs between surfaces (MCP `image` vs CLI
  `image_references`, recorded as unresolved in the higgsfield skill): confirm with `models_explore` before the call.
- "that image" / "that outfit" means the exact asset resolved from production status or clarified with the user; an art_ id or
  `--input <file>` names one exactly. Older project assets are never reused unless named.

Record exact source/draft/take IDs, settings and review state before handoff to `image-verify` or `video-from-image`; final promotion distinguishes enhancement/upscale from regeneration. If the `higgsfield` prompting skill or the Higgsfield connector is missing, install or connect it per `studio` → `references/dependencies.md`; a missing dependency is never a reason to substitute.

## OpenRouter images
Read `studio` → `references/openrouter.md` for environment checks, live models/prices, authorization and credit-cap conversion. Check the exact model's controls at `/images/models` and `/images/models/{author}/{model}/endpoints`. Write the prompt for that model, keeping the project's prompt rules; the Higgsfield prompting dependency applies only to Higgsfield.

After cost approval, run in the production folder with the selected prompt in `prompt.txt`; replace the example model with the resolved `models.image` choice:

```sh
jq -n --rawfile prompt prompt.txt --arg model 'openai/gpt-image-2' \
  '{model:$model,prompt:$prompt,n:1}' > image-request.json
curl --fail --silent --show-error https://openrouter.ai/api/v1/images \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" -H 'Content-Type: application/json' \
  --data-binary @image-request.json --output image-response.json
python3 - <<'PYIMAGE'
import base64, json
from pathlib import Path
response = json.loads(Path("image-response.json").read_text())
for index, item in enumerate(response["data"]):
    ext = {"image/png": "png", "image/jpeg": "jpg", "image/webp": "webp"}[item["media_type"]]
    Path(f"image-{index}.{ext}").write_bytes(base64.b64decode(item["b64_json"], validate=True))
PYIMAGE
```

Read `data[].b64_json` and `media_type` (if absent, establish the format before naming the file); record `usage.cost`. Persist decoded files with `asset-persist`, then `image-verify`. The example's `n:1` is one take; use the authorized count (API range 1-10), not extra automatic candidates.

Set supported `resolution` (512, 768, 1K, 1.5K, 2K, 4K), `aspect_ratio`, quality, output_format/background/compression and seed explicitly when chosen. Providers may clamp aspect ratios; inspect the result. An explicit pixel `size` conflicts with `resolution` / `aspect_ratio` and returns 400. Reference edits/composites use `input_references` (URLs or base64 data URLs; count/support varies by provider), at the same `/images` endpoint; no dedicated edit endpoint. Do not send Soul ids or Element placeholders. Verify reference/edit support live before promising identity preservation. A requested outfit asset is saved as a reference image here; creating a Higgsfield Element is a separate direct-provider action. Routing options under `provider` include `only`, `order`, `ignore`, `sort`, `allow_fallbacks`, `options`; preserve any pinned endpoint. Failed image generation returns 502 and is not billed (all-or-nothing); never treat a failure as approval to retry a paid request.
