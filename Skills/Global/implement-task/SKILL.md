---
name: implement-task
description: Execute one parked task from the project checklist to best practice, stopping at the commit gate.
disable-model-invocation: true
---

# Implement Task

Carries one parked task from `docs/CHECKLIST.md` to review-passed, working code, applying only the practices that fit the work.

Stop at the commit gate: do not commit or merge, and do not delete the task line. Do not write implementation before Step 2 has scoped what applies.

This skill is the single-task sibling of `implement-milestone`. Both sequence the pipeline defined in `development-workflow.md`; that rule, with `agents.md`, `testing.md`, and `code-review.md`, stays the source of truth for each stage. Point to them, do not relearn them here. A single task is smaller than a milestone, so the heavyweight stages (`planner`, `architect`, ADRs) usually do not apply — Step 2 decides.

## Step 1 — Load the task

Read `docs/CHECKLIST.md`. Select the open `[ ]` item by the number given on the command line (counting open items top to bottom). If no number was given, list the open items numbered and close with a single `Which? (1-N)`, then wait (global interaction-design doctrine). Ignore `[x]` items — those are done and awaiting commit.

The entry has no formal done-criteria the way a milestone does. Read the entry text and its `while:` context, then state a concrete done-criterion for the task — what must be true for it to count as finished. Read `CLAUDE.md` and the PRD (`docs/PRD.md`) for constraints that bind the work; product code lives under `product/` per `CLAUDE.md`'s Project Layout.

Done: one open task is selected, and you can state its scope, a concrete done-criterion, and the part of the codebase it touches.

## Step 2 — Scope what applies

Map the task to an approach, the same gate `implement-milestone` uses, sized for one task:

- **Test-first (`tdd` skill)** — applies wherever the task adds or changes behavior. Skip only where there is nothing to assert: docs, config, a trivial rename.
- **Reviewer agents** — the general `code-reviewer`, plus the stack's reviewer for each language touched (`go-reviewer`, `react-reviewer`, ...).
- **`security-reviewer`** — only when the work hits one of `code-review.md`'s security triggers (auth, user input, data access, secrets, crypto).
- **`planner` / `architect`** — normally not needed for a single task. Invoke `planner` only if the task turns out to span components; record an ADR only if a genuine hard-to-reverse decision arises.

Done: the task is mapped to test-first or direct implementation, and the agents that will run — and at which stage — are named.

## Step 3 — Research before building

Before writing new code, run the Research & Reuse step in `development-workflow.md`: search for an existing implementation, library, or pattern and prefer it over hand-rolled code.

Done: an existing approach is adopted, or none fits and that is confirmed.

## Step 4 — Build red-green

For every part Step 2 marked test-first, run the `tdd` skill: a failing test (red), the minimal code to pass (green), then refactor. Implement the rest directly against the done-criterion. Write product code under `product/` and run build/test commands from there. Hand a failing build to the stack's `*-build-resolver`.

Done: the task's done-criterion is met and the test and build commands pass — confirmed with evidence, not assumed.

## Step 5 — Review

Run the reviewer agents named in Step 2: `code-reviewer` and each stack reviewer, plus `security-reviewer` if flagged. Resolve every CRITICAL and HIGH finding before the gate; record any accepted MEDIUM or LOW item.

Done: reviews are clean, or their CRITICAL and HIGH findings are fixed and re-verified.

## Step 6 — Mark done, stop at the commit gate

Confirm the work against the done-criterion from Step 1. On verified completion, flip that entry in `docs/CHECKLIST.md` from `- [ ]` to `- [x]`, leaving the line and its `while:` context otherwise unchanged. Do not delete the line. Do not commit or merge.

Report the task as done and awaiting commit, and tell the user that `/wrap-it-up` will commit it and remove the line. Never invoke the milestone close from here — a single task does not close a milestone. If a milestone is currently in-progress, `/wrap-it-up` commits the task on its own (its boundary guard will not redirect to `/save-session` for a task commit); if a milestone is also completing, the task commit rides along with that wrap.

Done: the task is verified against its done-criterion, its checklist line reads `- [x]`, nothing is committed, and the user is pointed at `/wrap-it-up`.
