---
name: asset-persist
description: "Download a provider output URL (ElevenLabs flow URL that expires in ~2 h, Higgsfield rawUrl, OpenRouter video content) into the production folder, hash it, and record it in status.md. Use right after any generation."
---

# Asset persist

Run immediately after each provider job finishes. Expiring URLs are the reason this exists.

- **Takes:** a provider output URL or a local file.
- **Makes:** the file saved durably in the production folder, with its sha256.
- **Tool:** a download plus a hash. No other tooling.
- **Save:** put the file in `productions/<production-id>/` and add a line to `status.md` (source URL, provider job ID, hash, review state).

## Steps
1. Download to the production folder with a descriptive, ordered name (for example `scene03_take2.mp4`):
   ```sh
   curl -L --fail -o "<production folder>/<name>.<ext>" "<provider url>"
   ```
   Pick the extension from the provider's file type, not from guesswork. If the shell cannot reach the provider (blocked network), use the connector's download or ask the user to download it and give you the path; do not retry through other routes.
   For OpenRouter video, download by saved job id from the fixed API origin, with the key from the environment only:
   ```sh
   curl --fail --silent --show-error "https://openrouter.ai/api/v1/videos/$video_id/content?index=0" \
     -H "Authorization: Bearer $OPENROUTER_API_KEY" --output "<production folder>/<name>.mp4"
   ```
   Check HTTP success before recording the file. Do not log/share signed URLs or attach authorization to arbitrary output hosts. OpenRouter speech bytes and decoded images are local files; follow step 6.
2. Hash it: `shasum -a 256 <file>` (macOS) or `sha256sum <file>` (Linux).
3. If the same bytes are already in the production (same hash), do not save a second copy; link the new take to the existing file in `status.md`.
4. For OpenRouter, retain the job/generation id and fixed-origin endpoint instead of signed URLs (including in sidecars/status). Keep other provider URLs as a secondary, non-durable reference. The saved file is the durable one.
5. Mark the new file `pending_review`. Gates (QC, user) move it to `approved` or `rejected`; generated is never approved.
6. A local file (web-route download, FFmpeg output) is recorded by reference the same way; save it durably inside the project before recording it.

## Production handoff
Keep existing file names, IDs and lineage. Record in `status.md` or a linked sidecar the project/production, scene/beat/segment, order, draft/take/version, model/voice/character/seed/settings, provider job/node/result, measured placement/duration and review state, plus the parent files an output was made from. The coordinator records exact references and selected outputs in production status. For speech takes, record the actual render model in both the existing `<file>.json` sidecar (`model`) and `status.md`, following `studio` → `references/output-sidecar.md` and `references/production-handoff.md`. Follow `references/elevenlabs-tagging.md` → **Model changes and listening review** to flag tagged takes for re-listening on model changes. Do not infer identity or order from file names.

## Cleanup: keepsake rule
This is the one home of the cleanup rule; other skills point here. Disk pressure is solved with scratch hygiene (`studio` → `references/local-jobs.md`) and archiving, never by losing a source.
- **Never delete a source asset:** anything downloaded, generated or paid for, including old and rejected versions, music candidates, SFX, voice takes, generated stills and clips. Rejected and superseded files keep their state in `status.md`; move them to an archive folder inside the production if they clutter it.
- **Only these may be deleted:**
  1. exact duplicates by sha256: keep one copy, verify its hash first, and record which path was kept;
  2. local format conversions (a WAV made from an MP3, a proxy, a resized still) whose original is kept;
  3. rebuildable outputs: renders, mixes, previews, conversions and scratch copies that the kept sources and recorded settings can make again.
- **Check references before deleting.** Skip any file an NLE project file (`.fcproj`, `.prproj`, `.drp`, FCPXML, OTIO and the like) or a timeline/plan in the production references by path. Search the project files for the file name and its path first.
- **List, then act.** Write the candidate list (path, size, reason from the three above, sha256 for duplicates, "not referenced" checked) to `status.md` and show it to the user before deleting anything outside a scratch folder.
