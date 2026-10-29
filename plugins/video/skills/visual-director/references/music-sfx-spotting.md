# Music and SFX spotting against picture

The visual director spots cues; it does not render or mix them. The map goes to `audio-post-mix` (placement, ducking, swells, mix) and, for new cues, to `ai-music-and-sound` (brief) then `sound-render` or `suno`. An episode with picture is mixed in the editor with each track kept separate (`filmcraft` → "Assembly split") and its export measured with `audio-master`; an audio-only program is mastered by `audio-master`, once, on the finished mix.

## Track layout

| Track | Holds | Rule |
|---|---|---|
| A1 voice | the locked narration | unity gain, never edited, retimed or normalized in the plan |
| A2 music | music cues | never baked into A1 |
| A3 SFX | effects and sting hits | never baked into A1; each effect sits on a music cue or a picture event, never on dead air unless the project wants that |

These tracks become the editor's tracks, and the episode is mixed there (`filmcraft` → "Assembly split"); a pre-mixed file is a preview only. Overlapping cues in a crossfade take a second music track.

## Levels

Read `audio.mix_levels_db` from the project. Without it, starting values [observed in one plan, not yet mixed and measured]:
- music under speech, bed peaks and SFX levels: the ceilings in `audio-post-mix` → "Levels: beds breathe, peaks never overpower" (dense versus sparse SFX: "Sparse SFX" there); the mix is measured against them;
- swells +6 dB on an ordinary beat, +8 dB on the hook, a tease or the closing beat, never more than +10 dB;

## Placement rules

- **Cue in/out** sit in speech gaps or on a picture event (a cut, a sting hit), never under the middle of a word.
- **Swells live inside beat gaps**: start about 0.15 s after the last word, peak about 0.5 s later, release from about 0.40 s before the next word, and be back at bed level just before it, so no swell sits under a syllable.
- **Deliberate silence**: 1-2 s with no music before each twist (the turn, a tease, a reveal) unless the project says otherwise. A long silent stretch through the turn is a valid choice; flag it for the user's ear.
- **Hedge and caveat lines** (claims the script qualifies) run without music or on a very low bed, so the music does not sell what the words hedge.
- **Stings** (intro, outro, section bumpers) hit on their picture cut; their timing anchors (riser start, hit, title in) are written as times.
- **Every beat** is either a swell or a deliberate silence; write which.
- Cue lengths are checked against the source files (`ffprobe`); a cue shorter than its span is looped or crossfaded by the mixer, and the plan says so.
- Entries, exits, crossfades and dips follow `audio-post-mix` → "No sudden drops"; plan crossfades inside pauses, on a second music track.
- With only a few body SFX, follow `audio-post-mix` → "Sparse SFX": spot each on its picture event, ideally in a breather, and list their times for the reviewer.

## Spotting table (in `visual-plan.md`)

| Cue | Track | Source (src in) or brief | In | Out | Fades in/out s | Level | Covers | Swells (time, gain, label) | Holes (music off, why) |
|---|---|---|---|---|---|---|---|---|---|

Then one line listing all music-free stretches with their reasons, and one line mapping every beat to `swell` or `silence`. Licence, provenance and the exact credit line for each cue (receipt, model, terms) come from `ai-music-and-sound`'s shortlist ("Picking the music"); list any cue without them as a gap.
