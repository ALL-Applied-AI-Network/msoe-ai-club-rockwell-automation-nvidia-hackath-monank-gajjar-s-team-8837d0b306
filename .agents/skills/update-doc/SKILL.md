---
name: update-doc
description: Add a key detail to the team's Dataset 2 Word document (docs/Dataset2_Fluid_Mixing.docx) when the user explicitly asks for it to go into the doc, e.g. "add this to the doc", "put this in the doc", "update the doc with ...". Covers problem, dataset, analysis, solution, results, business case, decisions and open questions.
---

# Update Doc

`docs/Dataset2_Fluid_Mixing.docx` is the team-facing Word document for the project's focus challenge (Dataset 2: Fluid Mixing Station), from the problem to the final solution. Its audience is the team, mentors and anyone reading the repo or the final submission, so it must stay accurate, clear and self-contained.

Run this skill **only when the user tells you to add something to the doc**. Add exactly what the user asked for; do not add other details from the conversation on your own.

The user may also edit this file by hand in Word. **Always edit the existing file in place. Never regenerate it from scratch**, or their edits are lost.

## Section map

| Detail | Section |
|---|---|
| Hackathon logistics, schedule, judging, ground rules | 1. Hackathon Context |
| Problem statement and goal | 2. The Problem |
| Tanks, recipes, dispense order, line constraints | 3. How the Station Works (3.1–3.3) |
| Carryover behaviour and examples | 4. Colour Carryover |
| Existing Teledyne inspection | 5. Why the Current Inspection Is Not Enough |
| Data sources, formats, labels, links | 6. Data Provided (6.1 Links) |
| Organiser instructions and testing rules | 7. Organiser Guidance (7.1–7.4) |
| Our understanding of the problem, metric priorities | 8. Problem Understanding |
| Risks and pitfalls | 9. Potential Pitfalls |
| Solution ideas not yet chosen | 10. Candidate Solution Sketch |
| Chosen approach, models, baseline, results, limits, hardware test | 11. Our Solution (replace `TBD` cells) |
| Costs, savings, adoption | 12. Business Case (replace `TBD` cells) |
| Decisions | 13. Decision Log (new row: date, decision, reason) |
| Questions; remove one once answered and record the answer where it belongs | 14. Open Questions |
| New terms | 15. Glossary (new row) |

If no section fits, add a new Heading 2 subsection in the most logical place, or a new Heading 1 before the Decision Log.

## Procedure

1. Load the `anthropic-skills:docx` skill and follow its **"Editing existing documents"** workflow: unzip, run `merge_runs.py`, edit `word/document.xml`, re-zip, validate. Work in the session scratchpad, never inside `docs/`.
2. Before editing, confirm the file isn't open in Word (it would be locked). If it is, ask the user to close it.
3. Read the current content (unzip and read the XML, or `pandoc -t markdown`) and locate the target section.
4. Make the edit by **copying the XML of a neighbouring paragraph, bullet or table row** and changing its text, so new content inherits the existing styles:
   - headings use `Heading1` / `Heading2`;
   - bullets and numbered items keep their `w:numPr`;
   - tables keep the same cell widths and the shaded header row;
   - `**bold**`-style emphasis uses `<w:b/>` runs; field names like `vial_fill` use the Consolas font.
   Replace `TBD` when the detail fills it. Update facts that changed instead of adding contradicting lines. Mark the source where it matters ("organiser guidance" vs "team analysis").
5. Housekeeping on every update:
   - set **Last updated** in the title block to today's date;
   - update the **Status** text if the project stage changed;
   - add a row to **16. Change Log** (`date | what changed`).
6. Re-zip, validate, and write the result back over `docs/Dataset2_Fluid_Mixing.docx`.
7. If headings were added or renamed, refresh the table of contents with Word via PowerShell COM: open the file, `TablesOfContents | ForEach-Object { $_.Update() }`, save, close.
8. Check the result: export to PDF with Word COM (`ExportAsFixedFormat(path, 17)`, into the scratchpad), render the changed pages and look at them.
9. Never invent results, numbers or decisions. If a detail is uncertain, add it as an open question or mark it "unconfirmed".
10. Edit only `docs/Dataset2_Fluid_Mixing.docx`; create no other files in the repo. (This skill folder contains only SKILL.md.)
11. Reply with a brief summary of what was added and where.
