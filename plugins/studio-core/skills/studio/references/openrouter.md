# OpenRouter route

Route `openrouter` uses curl against `https://openrouter.ai/api/v1`. The user exports `OPENROUTER_API_KEY` in their shell environment, outside the repo and project file. Never ask for, print or store its value, read secret files, or use shell tracing / curl verbose output with authorization. Check only the exit status of `test -n "$OPENROUTER_API_KEY"` (never echo). Optional attribution headers are `HTTP-Referer` and `X-OpenRouter-Title`.

Resolve model and route from explicit production choices, else the project (`routes.openrouter`, `models.voice` / `models.transcription` / `models.image` / `models.video`, including `default`, `allowed`, `locked`). Missing keys mean no default. Use exact ids from live discovery; no silent swaps of models, voices or references. The live list is the source of truth, with no hard-coded model assumptions. [models/openrouter.json](models/openrouter.json) is only a seed/fallback when offline or the list is unreachable; label fallback choices as unconfirmed and recheck availability, controls and prices before generation.

## Discovery and paid-step preflight
`GET /api/v1/models` is public and non-billable: no key is needed for listing. Use `output_modalities=speech`, `transcription`, `image` or `video` (one per query), plus `/images/models` and `/videos/models` for modality-specific controls. This also works before a first project or key is configured; let the user pick defaults from the returned ids, or leave them `null` for "decide each time":

```sh
curl --fail --silent --show-error 'https://openrouter.ai/api/v1/models?output_modalities=speech' --output models.json
curl --fail --silent --show-error https://openrouter.ai/api/v1/images/models --output image-models.json
curl --fail --silent --show-error https://openrouter.ai/api/v1/videos/models --output video-models.json
```

Read the exact model's `pricing.prompt` / `pricing.completion` strings and their units from `/models`. TTS is usually per input character, sometimes per output second; STT may be per audio second or token. Do not apply text-token arithmetic to character prices. Image endpoint details at `GET /images/models/{author}/{model}/endpoints` expose `billable`, `unit`, `cost_usd`, `variant`; use them to check the `/models` estimate for the chosen settings. Video uses `/videos/models` `pricing_skus` (for example `cents_per_video_output_second_720p`): divide cents by 100 before multiplying by the applicable seconds/count. Missing price or ambiguous units means unknown, never zero; ask the user before spending. No estimate-only POST is documented here.

Record the USD estimate and its basis before approval, then actual `usage.cost` where returned (TTS returns raw bytes, so do not invent a usage object). Respect cumulative `spend.limit_credits` and `spend.ask_above_credits`: credits = USD × the explicitly set positive `spend.credits_per_usd`. Missing/null conversion means no default rate; ask the user to set one, or explicitly resolve the cap with a USD authorization before proceeding. Keep other providers' credit units identified; do not assume their credits are interchangeable. Price checks, existing spend authorization and review gates still apply; the examples below do not authorize a paid call. Submit once and poll rather than resubmitting slow jobs.

## ElevenLabs speech

The selected `voices[].model` supplies the model, never a code/provider default. Resolve explicit production choices into the profile for their scope; respect `models.voice` constraints. v4 is recommended by ElevenLabs for new work; selected v3 remains valid. Read [elevenlabs-tagging.md](elevenlabs-tagging.md) for detailed cited evidence and [models/openrouter.json](models/openrouter.json) for the snapshot. Evidence below is from the supplied official-source facts dated 2026-10-17; [observed] is the user's single A/B, not a call made in this edit.

[verified] `POST /api/v1/audio/speech` uses this shape (example, not a model default):

```json
{"model":"elevenlabs/eleven-v4","input":"[whispers] Stay close.","voice":"sarah","response_format":"mp3","provider":{"options":{"elevenlabs":{"apply_text_normalization":"auto"}}}}
```

- [verified] Bracket tag syntax is accepted in `input`; delivery is model-specific ([maintainer-observed]; see [model-change listening review](elevenlabs-tagging.md#model-changes-and-listening-review)). Tags count toward the character ceiling and billing per Unicode code point. Check upstream `models/elevenlabs.json` `tags_supported`; strip/convert tags for non-tag models. `voice` is one of 21 named voices, e.g. george/sarah/adam. [unverified] Direct-account voice-id portability. `response_format` is mp3 or pcm, default pcm.
- [verified] Omit `instructions`: docs say it is ignored except for OpenAI gpt-4o-mini-tts, Gemini TTS and Fish Audio. Put ALL ElevenLabs direction in tagged `input`.
- [observed] instructions: accepted without error but had no observable effect on elevenlabs/eleven-v4 in one A/B (2026-10-17); do not rely on it; output is nondeterministic so one test is indicative, not proof
- [verified] v4/v4 Turbo have no speed control and reject non-default speed; omit it. Speed 0.7-1.2 is documented only for Multilingual v2 and Flash, at `provider.options.elevenlabs.voice_settings.speed`; control v4 pace with tags/punctuation. [unverified] v3/Turbo v2 speed support here; omit unless established.
- [verified] `provider.options.elevenlabs` accepts `seed`, `previous_text`, `next_text` (~500 chars each way), `language_code` (ISO 639-1), `apply_text_normalization` (auto default), `apply_language_text_normalization`, `pronunciation_dictionary_locators`, `voice_settings` (`stability`, `similarity_boost`, `style`, `use_speaker_boost`, supported `speed`), `output_format`. Match codecs, e.g. mp3_22050_32 with response_format mp3 or pcm_16000 with pcm. Unlisted keys are ignored; invalid listed values return 400. [unverified] v3/v4 dictionary behavior.
- [verified] OpenRouter request ceilings: v4/v4 Turbo 10,000; v3 Conversational 5,000; Multilingual v2 10,000; Flash/Turbo v2.5 40,000; Flash/Turbo v2 30,000 characters. v4's model page gives no ceiling; the announcement supplies it. Failed generations are not billed.
- Repository policy: apply project `audio.chunking` / `audio.max_chunk_chars` below these ceilings, splitting at paragraph/sentence boundaries, never inside tags/sentences. v3 ~2,700 is a runbook target; v4/v4 Turbo 4,000 is an [inferred], untested repository target. [inferred] Repeat the active emotion tag per chunk; pass continuity context where supported. Direct v4 Turbo limit is [unverified]; the OpenRouter figure does not establish it.

Sources: [OpenRouter TTS](https://openrouter.ai/docs/features/multimodal/tts), [ElevenLabs announcement](https://openrouter.ai/blog/announcements/elevenlabs-on-openrouter), [v4 model page](https://openrouter.ai/elevenlabs/eleven-v4). The TTS docs have no ElevenLabs section; the announcement supplies the provider key names and speed/limit rules. Full A/B setup, direct-model guidance and ElevenLabs sources: [elevenlabs-tagging.md](elevenlabs-tagging.md).

## Transcription
For a transcript/caption step, preserve the selected STT model and use `POST /audio/transcriptions`. This JSON shape takes raw bytes encoded as base64, not a data URI; run in the production folder with the selected `audio.mp3`. Replace the example model only with the resolved choice, after cost approval.

```sh
python3 - <<'PY' > transcription-request.json
import base64, json
from pathlib import Path
print(json.dumps({"model": "openai/whisper-1", "input_audio": {
    "data": base64.b64encode(Path("audio.mp3").read_bytes()).decode(), "format": "mp3"},
    "response_format": "json"}))
PY
curl --fail --silent --show-error https://openrouter.ai/api/v1/audio/transcriptions \
  -H "Authorization: Bearer $OPENROUTER_API_KEY" -H 'Content-Type: application/json' \
  --data-binary @transcription-request.json --output transcription.json
```

Formats: `wav`, `mp3`, `flac`, `m4a`, `ogg`, `webm`, `aac`. Optional `language` is ISO-639-1, `temperature` is 0-1, and `provider` controls routing. Multipart is also accepted, with a 25 MB cap. Only `json` / `verbose_json` response formats are supported (`text`, `srt`, `vtt` return 400); `timestamp_granularities: ["segment", "word"]` requires `verbose_json`. `openai/gpt-4o-transcribe` and `microsoft/mai-transcribe-1.5` reject `verbose_json`. Response has `text` and `usage` (seconds/tokens/cost); verbose responses may add language, duration, segments and words, depending on provider. Split long audio for the 60-second upstream timeout; preserve offsets/order when joining. `elevenlabs/scribe-v2` and `scribe-v2-medical` are listed live but not documented on the STT guide; word/diarization options are unconfirmed. Review accuracy and timing before building captions; never invent word timestamps from plain text.

Sources (API shapes supplied/verified 2026-10-17): OpenRouter [TTS](https://openrouter.ai/docs/guides/overview/multimodal/tts), [STT](https://openrouter.ai/docs/guides/overview/multimodal/stt), [images](https://openrouter.ai/docs/guides/overview/multimodal/image-generation), [video](https://openrouter.ai/docs/guides/overview/multimodal/video-generation). Paid generation was not exercised in this repository change.
