---
name: save-for-later
description: Park a side-tracked task in the project checklist for later, without interrupting the work in hand.
disable-model-invocation: true
---

# Save For Later

Captures a side-tracked task into the project's `docs/CHECKLIST.md` so it is not lost, then returns to the task in hand.

This skill only appends and reports. Never plan, start, or implement the captured task — record it and hand control straight back to whatever was in progress.

## Step 1 — Capture the task text

Take the command argument as the task description. If it is empty, ask for one line and wait — do not infer a task from the surrounding work.

Done: a single, concrete task description is in hand.

## Step 2 — Infer the context

Derive a short `while:` phrase describing what was happening when the task was parked — the milestone, file, or problem in focus right now. One clause, drawn from the current conversation, no more. If the context is genuinely unclear, omit the `while:` line rather than guess.

Done: a one-clause context phrase exists, or it is deliberately omitted.

## Step 3 — Resolve the file

This is a same-project tool. Locate the project root (the directory holding `.claude/`). If there is no project here, say so and stop — do not create a global or cross-project list.

Ensure `docs/CHECKLIST.md` exists (create the `docs/` folder if the project predates it). If the file is absent, create it with a one-line header:

```markdown
# Checklist — parked tasks and tactical refinements
```

Done: `docs/CHECKLIST.md` exists, or the skill has stopped because there is no project.

## Step 4 — Append and report

Append one entry to the end of the file, never overwriting existing entries:

```markdown
- [ ] 2026-06-30 | auth/login error handling looks sketchy
      while: implementing the JWT cookie milestone
```

Use today's date. The `while:` line is indented under the entry; omit it if Step 2 omitted the context.

Report the parked task and the current open-item count in one line, then return to the task in hand. Do not act on the item.

Done: the entry is appended, the count is reported, and control has returned to the prior work.

## Relationship to other surfaces

`docs/CHECKLIST.md` holds granular, actionable tasks — side-tracks captured here and tactical refinements routed by `give-feedback`. It is distinct from docs/PROGRESS.md's "Deferred / future tasks", which holds milestone-scale future work set at wrap time. Keep them separate; do not copy items between them here.

- `continue-project` surfaces open `[ ]` items at session start and offers `/implement-task <n>` to execute one.
- `implement-task` executes a parked task and marks it `[x]` (done, awaiting commit).
- `wrap-it-up` commits completed `[x]` tasks and removes their lines. This skill never triages or commits.
