# Output sidecar

Every finished file gets a small JSON sidecar saved next to it, named `<output file>.json` (`episode.mp4` -> `episode.mp4.json`):

```json
{ "file": "episode.mp4", "kind": "final video", "status": "draft | pending_review | approved | rejected",
  "sha256": "…", "project": "<slug>", "production": "<id>", "made_with": ["filmcraft", "audio-master"],
  "created_at": "…", "notes": "",
  "credits": [{ "asset": "music/cue02_track.mp3", "title": "…", "author": "…", "licence": "CC BY 4.0",
                "credit_line": "exact text to paste", "where": "description" }] }
```

- `kind` is plain words (for example "final video", "mastered audio", "short"), not a type ID.
- The LLM writes it. Get the hash with `shasum -a 256 <file>`.
- `status` starts at `draft` or `pending_review`; only the user's review moves it to `approved` or `rejected`.
- `made_with` lists the skills used, by name.
- OpenRouter follows the same sidecar: `made_with` names the rendering skill; link job/model/route and USD cost details from the production record when needed. Never include key values, authorization headers or signed URLs.
- Speech takes use this same `<file>.json` sidecar: add `model` with the exact model id actually used to render that take, and record it per take in `status.md` too. Keep the render model even if the profile changes. Use `status: "pending_review"` and `notes` for tagged takes needing re-listening after a model change; see [elevenlabs-tagging.md](elevenlabs-tagging.md#model-changes-and-listening-review).
- `credits` carries every music, SFX or other third-party asset in the file that needs a credit or licence note, copied from the music shortlist (`ai-music-and-sound` → "Picking the music"): the exact `credit_line` text, its `licence`, and `where` it must appear (`description` for CC BY and most library licences). An empty list means nothing needs a credit; leave the key out only when the file has no music or SFX. A publishing tool reads it so the credit reaches the video description.
- Nothing validates it. Publishing can take the file with or without a sidecar.
