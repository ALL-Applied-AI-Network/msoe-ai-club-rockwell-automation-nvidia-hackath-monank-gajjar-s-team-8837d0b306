---
name: update-memory
description: Update the project memory skill (.agents/skills/memory/SKILL.md) after a crucial step. Use only when the user explicitly instructs it, e.g. "update memory", after finishing a milestone, decision, data finding or structure change.
---

# Update Memory

Updates `.agents/skills/memory/SKILL.md` so it reflects the project's real state. Run only when the user asks.

## Procedure

1. Read `.agents/skills/memory/SKILL.md` in full.
2. Determine what changed from the user's instructions and the conversation. If needed, check the repo (`git status`, `git log`, file tree) to confirm facts such as new files or folders. Do not run extra analyses.
3. Edit the relevant sections in place:
   - Idea / problem statement / data / tech stack: replace `TBD` with confirmed facts only.
   - Project Structure: redraw the tree to match the repo, with a short comment per file.
   - Progress Log: append one dated line (`YYYY-MM-DD - ...`) per crucial step; never delete past entries.
   - Key Decisions: add decisions with a brief reason.
   - Current Status and Open Questions / Next Steps: rewrite to the present state; remove resolved items.
4. Keep the existing section layout, the frontmatter and the `Modes` section intact. Do not add other files; only `SKILL.md` is allowed in a skill folder.
5. Record only facts that are known. Leave `TBD` where unknown, never guess. Never store secrets or raw data contents.
6. Reply with a short list of what was changed in memory.
