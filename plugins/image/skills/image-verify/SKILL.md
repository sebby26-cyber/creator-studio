---
name: image-verify
description: "Apply a project's image QC checklist to a still and emit a PASS / CONDITIONAL PASS / FAIL QC report. Generic framework: the checklist items, severities and reroll policy live in the project.json (qc.checklists)."
---

# Image verify

- **Takes:** a still image and the project's checklist name.
- **Makes:** a QC report (each item PASS / FAIL / N/A with a note, plus the verdict) and the image's review state.
- **Tool:** Claude viewing the image; no credits.
- **Save:** put the report in the production folder and add a line to `status.md`.

1. Load the checklist: `project.json` -> `qc.checklists.<name>` (the production or the request names it; see the
   project file for the available checklists). Read the matching gates in the project's
   `workflow.md`: they are verbatim and binding.
2. View the image itself (full size, then at the size the project tests at, e.g. 390 px). Judge every item
   PASS / FAIL / N/A with a one-line note naming what you saw.
3. Verdict: **PASS** only if every item passes; **CONDITIONAL PASS** only where the project allows it; else **FAIL**.
   Ambiguity is not a pass ("maybe fine" never moves downstream).
4. On FAIL, follow the project's reroll policy exactly (e.g. one targeted-fix regeneration with a corrective clause,
   max attempts, local patch instead of reroll for tiny artifacts). Every regeneration is a billable step: check its
   cost and get the OK unless it is already authorized.
5. Write the report {checklist, subject (file or artifact id), items[{id, criterion, verdict, note}], verdict} and update
   the asset status (`approved` / `rejected`, rejected files stay in the projects folder, never deleted).
6. User gates named in the project file are separate and always win: QC pass does not replace them.
