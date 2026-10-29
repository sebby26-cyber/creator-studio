# Output contracts: `visual-plan.md`, `timeline.json`, shot briefs

`clip-assemble` defines no timeline input format, and `filmcraft` names only the fields of a "measured shot/timing record" (shot IDs, `start_s`, `duration_s`, `total_s`, `timing_source: "speech"`, source IDs, source and program in/out). This file defines one shape that carries those fields, the `narration-and-story-scripting` shot-list fields, and everything the visual director decides. Schema id: `creator-studio.visual-timeline/1`.

All times are **seconds of program time**, measured from the first sample of the locked voice file unless `program.time_base` says otherwise (for example an intro offset). Three decimals are enough.

## Measuring the clock

Prefer the narration-edit timing table from `audio-post-mix`, or ASR word times aligned to the script. With neither, find the gaps and map them, in order, to the script's sentence ends [verified, FFmpeg 9.0.2]:

```sh
ffprobe -v error -show_entries format=duration -of default=nw=1:nk=1 voice.wav
ffmpeg -hide_banner -nostats -i voice.wav -af silencedetect=noise=-40dB:d=0.25 -f null - 2>&1 \
  | grep -oE "silence_(start|end): [0-9.]+"
```

- `silence_start` is the last word's end (+ release); `silence_end` is the next first word. Tune `noise` to the take's floor and `d` below the shortest designed rest.
- Not every gap is a sentence end (commas and breaths also make gaps), and some sentence ends run on with no gap: map gaps to sentences by order and expected length, and check each mapped boundary by ear or against ASR. ASR word times can drift by most of a second near short or repeated words [observed]; snap boundaries to the measured gap, not to the ASR time.
- Record the clock file, its sha256 and its measured duration in `audio_clock`.

## `timeline.json`

```json
{
  "schema": "creator-studio.visual-timeline/1",
  "status": "plan | approved | superseded",
  "project": "<slug>", "production": "<id>", "revision": 1, "date": "YYYY-MM-DD",
  "total_s": 312.480, "timing_source": "speech",
  "program": { "total_s": 312.480, "width": 1920, "height": 1080, "fps": 30, "frames": 9375,
               "aspect_ratio": "16:9", "time_base": "seconds from the first sample of the locked voice file" },
  "audio_clock": { "file": "<path>", "sha256": "<hash>", "duration_s": 312.480,
                   "timing_record": "<path to the measured sentence/word table>" },
  "rules": { "min_hold_s": 3.0, "max_hold_s": 10.0, "min_clip_speed": 0.6, "no_text": true, "text_allowed_windows": [[0, 5.9]],
             "source": "project | default (PROVISIONAL)" },
  "markers": {
    "beats":    [{ "id": "beat1", "rest_in": 11.28, "rest_out": 13.05, "line_before": "...", "line_after": "..." }],
    "turn":     { "id": "turn", "hold_in": 140.2, "hold_out": 142.6, "reveal_in": 142.6, "reveal_out": 146.1, "line": "..." },
    "reveals":  [{ "id": "R1", "at_s": 142.6, "before_state": "plain words", "after_state": "plain words" }],
    "sections": [{ "id": "sec1", "start_s": 0.0, "end_s": 58.7, "name": "hook" }]
  },
  "shots": [ { "...": "see the shot fields below" } ],
  "tracks": {
    "V2_titles": [{ "id": "T1", "start_s": 3.4, "end_s": 5.9, "source": "<path>", "text": "what it says" }],
    "A1_voice":  [{ "id": "VO", "file": "<path>", "start_s": 0.0, "end_s": 312.480, "gain_db": 0.0, "note": "locked" }],
    "A2_music":  [{ "id": "M1", "file": "<path or brief id>", "src_in_s": 0.0, "start_s": 4.6, "end_s": 60.5,
                    "fade_in_s": 1.5, "fade_out_s": 1.2, "level_lu_under_speech": 12, "swells": [], "holes": [] }],
    "A3_sfx":    [{ "id": "X1", "file": "<path or brief id>", "start_s": 0.0, "end_s": 3.5, "level_db": -10, "on_cue": "M1" }]
  },
  "transitions": [{ "at_s": 6.16, "type": "dissolve", "dur_s": 0.75, "between": ["S000", "S001"] }],
  "briefs": [ { "...": "see Shot brief" } ],
  "budget": { "cap_credits": 150, "ask_above_credits": 50, "currency_note": "credits; USD only via spend.credits_per_usd",
              "tiers": { "free": { "credits": 0 }, "minimal": { "credits": 50, "shots": ["S001"], "price_source": "reference 2026-10-08" },
                         "recommended": { "credits": 90 }, "full": { "credits": 170 } },
              "selected": null },
  "checks": { "script": "<the project's timeline checker, e.g. tools/check-timeline>", "result": "PASS | FAIL", "date": "YYYY-MM-DD" }
}
```

### Shot fields

| Field | Required | Meaning |
|---|---|---|
| `id` | yes | `S001`... in program order; stable across revisions (a split shot becomes `S014a`, `S014b`). |
| `start_s`, `end_s`, `duration_s` | yes | Program in/out. Shots are contiguous: each `start_s` equals the previous `end_s`. |
| `timing_source` | yes | `speech` (measured) or `estimate`. |
| `lines`, `narration` | yes | Sentence ids and the words under the shot (verbatim). |
| `first_word_s`, `last_word_s` | when measured | The speech inside the shot, for cut placement checks. |
| `markers` | when any | Marker ids this shot decides, e.g. `["beat3"]`, `["turn"]`, `["R1"]`. |
| `decision` | when markers | What happens at the marker: `hold`, `cut`, `static hold`, `dissolve`, `motion`. |
| `role` | yes | `establishing`, `detail`, `reveal`, `reaction`, `callback`, `bridge`, `sting`. |
| `source` | yes | `{kind: still|clip|sting|diagram|new_still|new_motion, id, path, native: "WxH", fps, baked_text: bool, origin, qc: PASS|CONDITIONAL PASS|FAIL|pending, interim: bool}`. A planned new asset has `path: null` and points to its brief. |
| `treatment` | yes | `{type: reuse|ken_burns|clip|new_still|i2v|morph|sting, move: {type: push_in|pull_out|pan_left|pan_right|tilt_up|tilt_down|parallax_fake|static, zoom: [from, to], anchor: "center" or [x, y]}, speed, src_in_s, src_out_s, grade, crop}`. |
| `method`, `description` | yes | Compatibility with the `narration-and-story-scripting` shot list: `method` is `still` (reuse, ken_burns, new_still), `animated` (clip, i2v) or `morph`; `description` is what the viewer sees. |
| `transition_in` | yes | `{type: cut|dissolve|fade_from_black, dur_s}`. |
| `brief_id` | for new assets | Points into `briefs[]`. |
| `continuity` | yes | `{state: {"R1": "before|after|neutral|transition"}, callback: bool, screen_direction, accuracy_flags: [], notes}`. `transition` marks a shot that carries the change itself (a morph or dissolve across the reveal); `callback: true` licenses a before-state image after its reveal when the narration points back to it. |
| `cost` | yes | `{credits: 0, tier: free|minimal|recommended|full, price_source: live|reference <date>|unknown}`. |
| `fallback` | for paid shots | The free treatment that plays if the paid one is not bought. |
| `hold_exception` | when outside the hold rules | One line saying why. |

### Shot brief

One per new still or new motion shot. The prompt-writer writes the prompt from this; the director never does.

```json
{ "id": "B03", "shots": ["S036"], "kind": "new_still | i2v | morph",
  "line": "the words it plays under", "time": [262.85, 269.16],
  "subject": "...", "framing": "wide / medium / detail; lens feel; composition",
  "lighting": "time of day, key direction, contrast",
  "look": "project look and grade constraints, quoted from the project rules",
  "motion_intent": "what moves, how far, camera move, needed duration (clips only)",
  "continuity": "state relative to each reveal; screen direction; which shot it must match",
  "references": ["start frame <id>", "end frame <id>", "style reference <id>"],
  "forbidden": ["any text or labels", "after-state objects", "..."],
  "target_model": "the project's locked/allowed model, or 'user to choose'",
  "prompting_method": "e.g. the higgsfield skill (project rule)",
  "interim": "asset id used until this exists",
  "why_paid": "which Step 6 criterion this meets" }
```

## `visual-plan.md`

Plain-language twin of the JSON, in this order:
1. Status line (PLAN ONLY, date, revision) and the audio clock (file, hash, duration).
2. Rules resolved (Step 1 table), with `NONE` / `PROVISIONAL` where the project is silent.
3. Marker table: beats, the turn, reveals (before/after state), sections.
4. Shot table: id, in, out, s, line(s), role, source or treatment, move, transition, brief id, continuity notes, cost.
5. Shot-brief list (the briefs above, readable).
6. Music/SFX spotting map ([music-sfx-spotting.md](music-sfx-spotting.md)).
7. Budget summary: counts per treatment per tier, credits per tier with price sources, the cap check.
8. Check report (the checker's output, verbatim).
9. Gaps and user decisions needed, numbered.

## Consumers

- **`clip-assemble` (FFmpeg)**: Ken Burns shots are rendered to clips first ([treatments-and-moves.md](treatments-and-moves.md)), so every V1 shot becomes a clip of exactly `duration_s`; then the "generated clips over the master audio" recipe concatenates them in `shots[]` order under a preview mix (a review preview, never the episode: `filmcraft` → "Assembly split"). A plain still with no move can use the slideshow recipe with `-loop 1 -t <duration_s>`. Check coverage before encoding: `-shortest` truncates.
- **`filmcraft` `edit_sequence`**: `program.width/height/fps` go to `file.newSequence`; each shot is a `timeline.place` on V1 at `seconds=start_s` with `duration = duration_s * 254016000000` ticks; transitions only where `transitions[]` says. The locked voice goes on A1, and `A2_music` / `A3_sfx` cues go on their own tracks at their times, mixed there with `audio-post-mix`'s levels (`filmcraft` → "Assembly split").
- **`audio-post-mix`**: reads `tracks.A2_music`, `tracks.A3_sfx` and `markers` as its cue list.
- **`image-generate` / `video-from-image`**: read `briefs[]` only through the prompt-writer.

The timeline checker (`SKILL.md` → Step 11) reads exactly the fields above; a consumer may ignore fields it does not use.

A small, fictional, complete example that passes every check is in [example-timeline.json](example-timeline.json) (a checker built to Step 11 must report PASS on it: use it as the checker's acceptance test). `rules.text_allowed_windows` is a list of `[start_s, end_s]` pairs; a shot or title passes only when it lies wholly inside one.
