---
name: give-feedback
description: Collect end-user observations about the app, then route each to a requirement change, a defect, or a tactical refinement — updates docs and plans only, never implements.
disable-model-invocation: true
---

# Give Feedback

Captures end-user observations about the application and routes each one to the right action. Produces updated documentation and plans. Implementation is a separate step.

Do not implement anything during this skill. Do not route feedback before collecting it all.

## You are routing, not fixing

Every item leaves this skill as a doc update, a plan change, or a parked task — never as a code change. Understanding a fix is expected: read the code, confirm what the feedback really is, and see that the change is one line if it is. Applying that fix, or offering to, is the leak. It bites hardest while collecting, when the code is fresh and the change looks trivial — that is exactly the moment a feedback pass turns into a lost session.

Red flags — stop the moment you notice one:
- Offering to fix: "want me to quickly fix this while I'm here?"
- Starting to write or apply the change, or sketching the diff.
- Treating a defect as too trivial to park.

Reading the code to understand or reframe an item is not a red flag — it is the work.

If the user asks for the fix now ("can you just fix it?"), you cannot pretend a skill overrides them — but this skill still does not implement. Park it in `docs/CHECKLIST.md` and point to `/implement-task <n>`, which runs it test-first and reviewed in its own session. State that, and leave the choice to them.

Worked example:
> User: "That button does nothing — small bug, just fix it while you're here."
> Right: "That is a defect — logged in docs/CHECKLIST.md as item 4. I will not fix it inside this feedback pass; `/implement-task 4` will do it test-first in its own session once we are done. Anything else about the app?"
> Wrong: "Sure, that is a one-liner." (then implements)

## Step 1 — Collect and interpret

Work item by item, at the user's pace. For each observation the user raises, let them finish describing it, then investigate the code to confirm what it really is — the reported symptom may have a different root cause than the user imagined, which can reframe the item or change which category it lands in later. State its technical implication: what would need to change for the feedback to be addressed. Understanding the fix is the work; applying it or offering to is not (see "You are routing, not fixing").

Continue until the user signals every observation is in. Do not categorize, consolidate, or route anything yet.

Done: every item has been gathered and interpreted, with its technical implication stated, and none has been categorized, consolidated, or routed.

## Step 2 — Categorize

Once all feedback is in, categorize the whole set in one pass — not item by item as it arrived — so the full picture is in view. Each item is one of:
- Requirement change: changes what the product does or is required to do
- Defect: the product already ought to do this and does not — a bug against an existing requirement; it needs no PRD edit, only a parked fix
- Tactical change: adjusts how something looks, reads, or feels — does not change what the product does

When unclear, ask. Do not guess the category.

Done: every item is labeled a requirement change, a defect, or a tactical change, in a single pass over the full set, with ambiguous ones resolved by asking.

## Step 3 — Consolidate

With every item categorized and the full set in view, merge genuine same-work items: several observations that describe one piece of work — the same fix, the same screen or component — become a single item carrying the combined intent. Merge only within one routing destination; keep genuinely distinct items separate.

Present the consolidated set to the user as a confirmation gate: what will be routed where, grouped by category (bulleted), then `Proceed? (Y/N)` (global interaction-design doctrine). Wait for confirmation before routing.

Done: same-work items are merged, distinct items kept separate, and the consolidated overview is confirmed by the user before any routing.

## Step 4 — Requirement path

For each requirement change:
a. Update docs/PRD.md to reflect the new or modified requirement.
b. Review every remaining milestone in the Delivery Milestones table. For each one, determine: does this change affect what this milestone must deliver? Milestones may need to be updated, split, removed, or added — a requirement change can require a new milestone of its own.
c. Evaluate the change against ADR criteria:
   - Hard to reverse
   - Surprising without context
   - A real trade-off with genuine alternatives
   If all three apply, draft an ADR in docs/adr/ before updating the milestone plan.
d. Present the updated milestone plan as a confirmation gate: what changed (bulleted), then `Proceed? (Y/N)` (global interaction-design doctrine). Wait for confirmation before any implementation begins.

Done: docs/PRD.md and the milestone plan reflect every requirement change, any qualifying ADR is drafted, and the updated plan is presented and awaiting confirmation.

## Step 5 — Parked-fix path (tactical changes and defects)

Each tactical change and each defect becomes a parked task in `docs/CHECKLIST.md`, the same surface and format `save-for-later` uses — so `continue-project` surfaces it and `/implement-task <n>` can execute it later. A defect is parked here, not fixed now; the fix is a separate, later session.

Ensure `docs/CHECKLIST.md` exists (create the `docs/` folder if the project predates it). If the file is absent, create it with the one-line header `save-for-later` uses:

```markdown
# Checklist — parked tasks and tactical refinements
```

Append one entry per item to the end of the file, never overwriting existing entries, in `save-for-later`'s format — a dated `[ ]` line with an indented `while:` context clause naming the feedback origin:

```markdown
- [ ] 2026-07-06 | results screen empty state reads as an error, not "no data yet"
      while: end-user feedback on the results screen
- [ ] 2026-07-06 | export button does nothing on click — should download the CSV
      while: end-user feedback on the export screen
```

Use today's date. Keep the format identical to `save-for-later` so both writers and every reader stay in sync; if that format changes, change it in one place. A tactical change too large for a single checklist task becomes a new milestone in the Delivery Milestones plan instead of a checklist entry — it needs no PRD change, since it does not change what the product does (a broad design-polish pass is the typical case).

Done: every tactical change and every defect is an open `[ ]` entry in `docs/CHECKLIST.md` in `save-for-later`'s format, except any too large for one task, which is a milestone in the plan.

## What this skill does not do

It produces updated docs/PRD.md, an updated milestone plan, and parked tactical tasks and defects in `docs/CHECKLIST.md`. The user confirms the milestone plan; the parked tasks are executed later via `/implement-task`. Implementation happens in a subsequent session.
