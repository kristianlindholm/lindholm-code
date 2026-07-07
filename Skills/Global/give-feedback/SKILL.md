---
name: give-feedback
description: Collect end-user observations about the app, then route each to a requirement change or a tactical refinement — updates docs and plans only, never implements.
disable-model-invocation: true
---

# Give Feedback

Captures end-user observations about the application and routes each one to the right action. Produces updated documentation and plans. Implementation is a separate step.

Do not implement anything during this skill. Do not route feedback before collecting it all.

## Step 1 — Collect

Gather all feedback before categorizing or acting on any of it. If the user is describing the app, let them finish.

Done: every feedback item has been gathered, and none has been categorized or acted on yet.

## Step 2 — Interpret

For each item, establish its technical implication: what would need to change in the codebase for this feedback to be addressed?

Done: each item has a stated technical implication.

## Step 3 — Categorize

Each item is one of:
- Requirement change: changes what the product does or is required to do
- Tactical change: adjusts how something looks, reads, or feels — does not change what the product does

When unclear, ask. Do not guess the category.

Done: every item is labeled a requirement change or a tactical change, with ambiguous ones resolved by asking.

## Step 4 — Requirement path

For each requirement change:
a. Update docs/PRD.md to reflect the new or modified requirement.
b. Review every remaining milestone in the Delivery Milestones table. For each one, determine: does this change affect what this milestone must deliver? Milestones may need to be updated, split, or removed.
c. Evaluate the change against ADR criteria:
   - Hard to reverse
   - Surprising without context
   - A real trade-off with genuine alternatives
   If all three apply, draft an ADR in docs/adr/ before updating the milestone plan.
d. Present the updated milestone plan. Wait for confirmation before any implementation begins.

Done: docs/PRD.md and the milestone plan reflect every requirement change, any qualifying ADR is drafted, and the updated plan is presented and awaiting confirmation.

## Step 5 — Tactical path

Each tactical change becomes a parked task in `docs/CHECKLIST.md`, the same surface and format `save-for-later` uses — so `continue-project` surfaces it and `/implement-task <n>` can execute it later.

Ensure `docs/CHECKLIST.md` exists (create the `docs/` folder if the project predates it). If the file is absent, create it with the one-line header `save-for-later` uses:

```markdown
# Checklist — parked tasks and tactical refinements
```

Append one entry per tactical change to the end of the file, never overwriting existing entries, in `save-for-later`'s format — a dated `[ ]` line with an indented `while:` context clause naming the feedback origin:

```markdown
- [ ] 2026-07-06 | results screen empty state reads as an error, not "no data yet"
      while: end-user feedback on the results screen
```

Use today's date. Keep the format identical to `save-for-later` so both writers and every reader stay in sync; if that format changes, change it in one place.

Done: every tactical change is an open `[ ]` entry in `docs/CHECKLIST.md` in `save-for-later`'s format.

## What this skill does not do

It does not implement changes. It does not make implementation decisions. It produces updated docs/PRD.md, an updated milestone plan, and parked tactical tasks in `docs/CHECKLIST.md`. The user confirms the milestone plan; the tactical tasks are executed later via `/implement-task`. Implementation happens in a subsequent session.
