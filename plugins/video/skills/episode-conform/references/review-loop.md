# Review loop

Two roles: the **builder** (changes tables, rebuilds, runs QC, writes `status.md`) and the **producer** (reviews the export independently and writes the fix list). They can be the same agent in separate passes, but the producer never takes the builder's status on trust: every claim it relies on, it measures or watches again. The user holds the final gate; no agent records an approval.

## 1. Producer review
`review/producer-notes-r<N>.md`, in this order:
1. **What was reviewed:** the export's hash, frame count and duration; what was measured independently; the tool discipline (one job at a time, local scratch).
2. **Verdict:** numbered points, plain language, the whole film watched and listened to on headphones and speakers.
3. **Fix list,** each item ranked:
   - **P1** blocks the user's gate (a cut-off word, a late duck heard as the voice cutting out, a sync error, a frozen or black frame, a ceiling breach under speech, a failed null test, a caption off its words);
   - **P2** a level, timing or picture mismatch against the plan or the rules (a lift off target, an SFX that does not read, a repeat over the flags, a crop under the floor);
   - **P3** polish or completeness (a marker missing, an evidence file not saved, a nicer move).
   Each item has **Where** (time, shot, cue), **What's wrong** (measured), **Layer and shape** (from the triage, section 2), **Fix** (the table row and the editor keys and values) and **Verify** (the numeric pass, with the QC checklist line).
4. **Re-export note for the builder:** the invariants that must hold after the fixes (frame count, captions identical unless timing changed, null tests, regenerated hash, sidecar and status).
5. **Checked and left alone:** what was deliberately not changed, and why.
6. **User decisions:** anything that changes a rule, a creative choice or spend, kept separate. The builder does not act on these without the user's yes.

From the second round, add an **"r<N−1> fixes: verified"** table (item, measured result, PASS/FAIL), a re-check of the delivery invariants, a verdict on the builder's judgement calls (accepted or not), and a final full-film pass.

## 2. Triage before fix
A reported defect (the user's note, a producer's item, a failed check) is located before anything is changed. A fix chosen before the layer is known can improve the wrong thing and leave the defect in place for several versions (a fade lengthened while the gap it led into stayed digital silence).
1. **Take the report as given:** the user's own words and the timecode on the export.
2. **Measure each layer at that point:** the export, the isolated voice stem, the bed (music and SFX stems), and the raw take, each at the same moment mapped onto its own clock (program time → voice-lock time → parent-take time through the insert table; data contracts → "Voice inserts"). A layer measured on the wrong clock proves nothing.
3. **Classify** the layer (source take, insert or edit, assembly or level, export or encode, playback only) and the shape (a **hole**: digital silence or a drop to nothing; a **dip**: a short fall in level; a **glitch**: a click, step or skip; a **level** problem: too loud or soft against its neighbours). Look for the shape the user described, not only the one the last scan looked for. A defect in the file but not in the editor's playback, or the reverse, is told apart per `clip-assemble` → "Playback stutter versus a file fault".
4. **Then fix**, at that layer, as a table row (section 3). Re-check the fix at the **same points** on the new export and on each layer before the item is closed, and record the before and after in the "verified" table.

Heavy measurement goes to a subagent with a fixed brief: the timecodes and file paths in, a per-layer verdict table out (timecode | export | voice stem | bed | raw take | layer | shape | evidence file), one job at a time under `studio` → `references/local-jobs.md`. The builder acts on the table, not on a summary.

## 3. Fix, rebuild, re-review
1. The builder applies every P1 and P2 as table rows in a **new version folder** (SKILL.md → "Rebuild method"), rebuilds in one pass, runs the full QC checklist, and writes `status.md` with sections for what changed, judgement calls and open user calls. P3 items are fixed when cheap or listed as open.
2. The producer re-reviews the new export.
3. Repeat until **no P1 or P2 remains**. Then the producer's verdict is "ready for the user's gate".
4. Spend is reported every round (credits and money, unknown amounts labelled). An item that needs spend or a rule change is quoted and waits for the user's OK.

## 4. User's gate
The user watches the export and decides. Until then the export's sidecar says draft (`pending_review`) and `status.md` lists the open user decisions. The user's notes start the next round as a new fix list.
