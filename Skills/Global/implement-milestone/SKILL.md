---
name: implement-milestone
description: Execute the active milestone to best practice — scope what applies, then research, plan, build test-first, and review, stopping at the commit gate.
disable-model-invocation: true
---

# Implement Milestone

Carries the active milestone from its `docs/PROGRESS.md` scope to review-passed, working code, applying only the practices that fit the work.

Stop at the commit gate: do not commit or merge. Do not write implementation before Step 2 has scoped what applies.

This skill sequences the pipeline defined in `development-workflow.md`; that rule, with `agents.md`, `testing.md`, and `code-review.md`, stays the source of truth for each stage. Point to them, do not relearn them here.

## Step 1 — Load the milestone

Read the active milestone from `docs/PROGRESS.md`: its scope, its done-criteria, and the `RESUME HERE` pointer if present. Read `CLAUDE.md` and the PRD (`docs/PRD.md`) for constraints that bind the work. Note from `CLAUDE.md`'s Project Layout that product code lives under `product/`.

Done: you can state the milestone's scope, its done-criteria, and the parts of the codebase and stack it touches.

## Step 2 — Scope what applies

Map every part of the milestone's scope to an approach. This gate decides the rest of the run:

- **Test-first (`tdd` skill)** — applies wherever the milestone adds or changes behavior. Skip only where there is nothing to assert: docs, config, scaffolding, throwaway spikes.
- **Design (`frontend-design` skill)** — applies *only* where the milestone adds or changes user-facing UI. When it does, the project's `docs/DESIGN.md` is the binding constraint; screens are derived from it, not reinvented. If `docs/DESIGN.md` is absent (Gate 3 was deferred, or an older project), create it first — run `frontend-design` to a signed-off styleboard and `docs/DESIGN.md` before building any UI (see `new-project` Gate 3). **Explicit skip:** a milestone with no user-facing UI — backend, API, domain logic, data, infrastructure — runs no design step, no sign-off, and no design review. It proceeds exactly as it would without this skill, with zero design overhead.
- **Reviewer agents** — the general `code-reviewer`, plus the stack's reviewer for each language touched (`go-reviewer`, `react-reviewer`, ...).
- **`security-reviewer`** — only when the work hits one of `code-review.md`'s security triggers (auth, user input, data access, secrets, crypto).
- **`planner` / `architect`** — `planner` when the milestone is large or spans components; `architect`, plus an ADR, when a genuine hard-to-reverse decision arises (new-project's three ADR criteria).

Done: every part of the scope is mapped to test-first or direct implementation, any user-facing UI is flagged for the design pass (or the milestone is confirmed to have none), and the agents that will run — and at which stage — are named. Nothing in scope is unaccounted for.

## Step 3 — Research before building

Before writing new code, run the Research & Reuse step in `development-workflow.md`: search for existing implementations and prefer a proven library or pattern over hand-rolled code.

Done: for each non-trivial part, you have either adopted an existing approach or confirmed none fits.

## Step 4 — Plan if Step 2 flagged it

If Step 2 called for planning, invoke the `planner` agent for an ordered, phased plan, and record any genuine architectural decision as an ADR. Otherwise state that the milestone is small enough to implement directly.

Done: a confirmed plan exists, or planning is explicitly waived with its reason.

## Step 5 — Design the UI surfaces (only if Step 2 flagged UI)

Skip this step entirely for a milestone with no user-facing UI.

When the milestone touches UI, run the `frontend-design` skill against the project's
`docs/DESIGN.md` to design the specific screens this milestone adds — layout and composition
derived from the approved visual language, not re-decided. Present the proposed screen design
for a hard sign-off (a rendered view where the environment allows it) and wait for the user's
explicit approval before implementing. If `docs/DESIGN.md` does not yet exist, establish it first
per Step 2.

Done: the milestone's UI screens are designed against `docs/DESIGN.md` and the user has signed off,
or the milestone has no UI and this step was skipped.

## Step 6 — Build red-green

For every part Step 2 marked test-first, run the `tdd` skill: a failing test (red), the minimal code to pass (green), then refactor. Implement the remaining parts directly against the done-criteria. Write all product code — source, tests, and the project manifest — under `product/`, and run build/test commands from there. Hand a failing build to the stack's `*-build-resolver`.

Done: the milestone's done-criteria are met and the test and build commands pass — confirmed with evidence, not assumed.

## Step 7 — Review

Run the reviewer agents named in Step 2: `code-reviewer` and each stack reviewer, plus `security-reviewer` if flagged. Resolve every CRITICAL and HIGH finding before the gate; record any accepted MEDIUM or LOW item.

If Step 2 flagged UI, also run a design review: screenshot the built screen (via the `run` or
`verify` skill where the environment supports it) and check it for conformance to `docs/DESIGN.md`,
plus the `design-quality.md` (aesthetics) and `design-principles.md` (craft floor) rules —
verified visually, not asserted. For a backend-only milestone this review does not run.

Done: reviews are clean, or their CRITICAL and HIGH findings are fixed and re-verified.

## Step 8 — Stop at the commit gate

Do not commit or merge — that ritual belongs to `wrap-it-up`. Confirm the milestone against its done-criteria, then recommend the next move:

- **Done-criteria met** → recommend `/wrap-it-up` to reconcile the docs, perform the git ritual, and open the next phase.
- **Work remains, or this is mid-task** → recommend `/save-session` to capture the handoff.

Done: the milestone is reported as complete or partial against its done-criteria, and the matching closer is recommended.
