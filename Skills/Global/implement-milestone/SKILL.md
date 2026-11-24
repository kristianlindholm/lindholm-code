---
name: implement-milestone
description: Execute the active milestone to best practice — scope what applies, then research, plan, build test-first, and review, stopping at the commit gate.
disable-model-invocation: true
---

# Implement Milestone

Carries the active milestone from its `docs/PROGRESS.md` scope to review-passed, working code, applying only the practices that fit the work.

Stop at the commit gate: do not commit or merge. Do not write implementation before Step 2 has scoped what applies.

This skill is for a product project. If `.claude/wrap-it-up.json` records `"mode": "prototype"`, stop and use `/build-prototype` instead — it runs the whole milestone plan in one continuous pass without the gates below. After `/promote-prototype` has published the project, this skill is the right one again.

This skill sequences the pipeline defined in `development-workflow.md`; that rule, with `agents.md`, `testing.md`, and `code-review.md`, stays the source of truth for each stage. Point to them, do not relearn them here.

## Step 1 — Load the milestone

Read the active milestone from `docs/PROGRESS.md`: its scope, its done-criteria, and the `RESUME HERE` pointer if present. Read `CLAUDE.md` and the PRD (`docs/PRD.md`) for constraints that bind the work. Note from `CLAUDE.md`'s Project Layout that product code lives under `product/`.

`CLAUDE.md` is an index: it carries the rules binding every change, and a `## Rules` table pointing at the topic files holding the rest. **Do not read those files yet, and do not read them all** — Step 2 decides which ones this milestone needs. Reading every topic file defensively costs more than the old single-file arrangement and buys nothing.

Done: you can state the milestone's scope, its done-criteria, and the parts of the codebase and stack it touches.

## Step 2 — Scope what applies

Map every part of the milestone's scope to an approach. This gate decides the rest of the run:

- **Test-first (`tdd` skill)** — applies wherever the milestone adds or changes behavior. Skip only where there is nothing to assert: docs, config, scaffolding, throwaway spikes.
- **Design (`frontend-design` skill)** — applies *only* where the milestone adds or changes user-facing UI. When it does, the project's `docs/DESIGN.md` is the binding constraint; screens are derived from it, not reinvented. If `docs/DESIGN.md` is absent (Gate 3 was deferred, or an older project), create it first — run `frontend-design` to a signed-off styleboard and `docs/DESIGN.md` before building any UI (see `new-project` Gate 3). **Explicit skip:** a milestone with no user-facing UI — backend, API, domain logic, data, infrastructure — runs no design step, no sign-off, and no design review. It proceeds exactly as it would without this skill, with zero design overhead.
- **Reviewer agents** — the general `code-reviewer`, plus the stack's reviewer for each language touched (`go-reviewer`, `react-reviewer`, ...).
- **`security-reviewer`** — only when the work hits one of `code-review.md`'s security triggers (auth, user input, data access, secrets, crypto).
- **Build-resolver** — the stack's `*-build-resolver` for each language touched (`go-build-resolver`, `react-build-resolver`, ...), for Step 7. Name it here even though it may not be needed, so Step 7 has a list rather than a search. Not every language has one and not every project provisions one: if `.claude/agents/` holds no resolver for this stack, record that here and Step 7 clears the build without a dispatch.
- **`planner` / `architect`** — `planner` when the milestone is large or spans components; `architect`, plus an ADR, when a genuine hard-to-reverse decision arises (new-project's three ADR criteria).

**Then name the rules this scope selects — in writing, before opening any of them.** Read `CLAUDE.md`'s `## Rules` index and `docs/adr/README.md`, then write the list: every file you are about to open and the practice, subsystem or `Binds` value that selected it. Only then open them, and open exactly that list.

**The index is the only way you find these files.** Do not list, glob or search `docs/rules/`, `docs/architecture/` or `docs/adr/` — a sweep of those directories reads every file in them, leaving you to select after the fact instead of before it. If a file is not in the index it is not yours to open. If the index names a file that does not exist, report it and carry on without it; do not go looking for a renamed one.

An older ADR index may list entries by number and title with no filename. Opening `docs/adr/<NNNN>-*.md` by its number is the one lookup this rule allows, since the slug cannot be derived from the title — note the gap so the index gains a `File` column.

Naming the set first is what makes it a decision. A list written afterwards is a description of what you happened to read.

- Mapped to test-first → the `testing` rows.
- Hit a security trigger → the `security` rows.
- User-facing UI → the design rows (`docs/DESIGN.md` already binds; open any design rule rows beside it).
- Touching a named subsystem → `docs/architecture/<subsystem>.md`.
- **Topics that no Step 2 practice selects — `coding-style`, `performance`, `patterns` — are selected by the code instead.** Open one when the milestone changes code of that kind: a hot path or a new input bound opens `performance`; adding or growing files opens `coding-style`, which is where a project records a file-size cap that is not the global 800. This rule exists because these three bind real work and nothing in the practice mapping above would ever reach them.
- Everything else → **skip it.** That skip is the whole point: the index exists so the weight is paid only where it applies.

Done: every part of the scope is mapped to test-first or direct implementation, any user-facing UI is flagged for the design pass (or the milestone is confirmed to have none), and the agents that will run — and at which stage — are named. Nothing in scope is unaccounted for. **The selected list was written before anything was opened, what you opened matches it exactly, no rules directory was swept, and you can name what you deliberately skipped.**

## Step 3 — Research before building

Before writing new code, run the Research & Reuse step in `development-workflow.md`: search for existing implementations and prefer a proven library or pattern over hand-rolled code.

Done: for each non-trivial part, you have either adopted an existing approach or confirmed none fits.

## Step 4 — Plan if Step 2 flagged it

If Step 2 called for planning, invoke the `planner` agent for an ordered, phased plan, and record any genuine architectural decision as an ADR. Otherwise state that the milestone is small enough to implement directly. Invoking this skill is the request for the `planner` agent — dispatch it without re-asking.

Done: a confirmed plan exists, or planning is explicitly waived with its reason.

## Step 5 — Design the UI surfaces (only if Step 2 flagged UI)

Skip this step entirely for a milestone with no user-facing UI.

When the milestone touches UI, run the `frontend-design` skill against the project's
`docs/DESIGN.md` to design the specific screens this milestone adds — layout and composition
derived from the approved visual language, not re-decided. Present the proposed screen design
for a hard sign-off (a rendered view where the environment allows it) and wait for the user's
explicit approval before implementing. If `docs/DESIGN.md` does not yet exist, establish it first
per Step 2.

**Then resolve the interaction decisions for each surface this milestone adds**, before
implementing and as part of the same sign-off. These are the questions in
`design-principles.md`'s Interaction decisions section, scoped by the project's usage context in
`docs/DESIGN.md`. Ask them one at a time, per the interaction-design doctrine:

- **Keyboard paths.** For each action on this surface, is there a keyboard route a daily operator
  would want? Delete on a selected item, Enter to submit, Escape to close, arrow keys within a
  list, a shortcut for the most-used action.
- **Focus on state change.** When a modal, drawer, or panel opens, where does focus go, and where
  does it return on close?
- **Control sizing.** Which controls are hit most often, and are they sized and placed for it?
- **Presentation robustness** — only when the usage context includes a projector, a client's
  screen, or an unfamiliar resolution.

Any answer is valid, including "no, not worth it for this surface". Record every answer in
`docs/DESIGN.md` under Interaction decisions — including the negatives, so the question is not
re-asked and the decision stays retrievable. Leaving one blank is the failure mode this step
exists to prevent: a surface ships with a delete button and no Delete key not because anyone
decided against it, but because nothing asked.

Done: the milestone's UI screens are designed against `docs/DESIGN.md`, the interaction decisions
for each new surface are answered and recorded, and the user has signed off — or the milestone has
no UI and this step was skipped.

## Step 6 — Build red-green

For every part Step 2 marked test-first, run the `tdd` skill: a failing test (red), the minimal code to pass (green), then refactor. Implement the remaining parts directly against the done-criteria. Write all product code — source, tests, and the project manifest — under `product/`, and run build/test commands from there.

Compile and type errors met along the way are part of the cycle; fix them and carry on. Do not dispatch an agent mid-cycle — Step 7 is where a build that will not come green is handed off.

Done: the milestone's done-criteria are met, and the build and test commands have been run — whatever they report.

## Step 7 — Clear the build

Run the build, typecheck and test commands now, and read the output.

**Green — say so and go to Step 8.** Nothing to dispatch.

**Red — dispatch the build-resolver named in Step 2 over the failures.** Invoking this skill is the request for that agent; dispatch it rather than fixing the build yourself. Give it the failing command and its output, scoped to this milestone's work. When it returns, re-run the commands yourself: its report that the build is fixed is not evidence, the command is.

**Red, and Step 2 recorded no resolver for this stack** — fix it yourself and say so. Most projects have no build-resolver provisioned, and several supported languages have none in the store at all; that is a coverage decision, not a reason to stall.

Clearing the build is its own step, not a clause inside Step 6. Do not fold it back in: a dispatch asked for mid-cycle does not happen, and one you arrive at does.

If the resolver cannot fix it, stop and report. A build that will not come green is not a milestone that is nearly done.

Done: the build, typecheck and test commands pass — confirmed by running them after any dispatch, not by an agent's report.

## Step 8 — Simplify

Dispatch `code-simplifier` over this milestone's diff — invoking this skill is the request for that agent, so dispatch it without re-asking. Scope it to the code this milestone touched, never the wider codebase, and treat "nothing worth changing" as a normal result.

**Then re-run the test and build commands.** The agent's claim to have preserved behaviour is not evidence; the suite is. If anything fails, restore from the `<file>.agent-bak` copies its own working-tree procedure requires — never with git.

Its edits are ordinary working-tree changes and go to Step 10's review with the rest of the diff. Running it here, before Step 10 snapshots the tree, keeps those writes inside the baseline rather than making them a divergence no agent appears to have caused.

Done: `code-simplifier` has had its pass and the test and build commands still pass.

## Step 9 — Run it

Invoke the `verify` skill: start the application and read what the running system says. Invoking this skill is the request for it — run it without re-asking.

**This step reports; it does not block.** A verify finding is information for the review that follows, not a gate on the milestone. Where verify reports "not ready to verify" — an absent dependency, a missing declared key — that is a setup gap, not a defect in this milestone's work.

It runs before the review rather than after, for two reasons. A startup failure gets fixed inside the diff the reviewer then reads, so one review pass covers both. And starting an application writes build artifacts into the working tree, while Step 10 opens by snapshotting the tree and comparing before-figures — running verify first makes those writes part of the baseline rather than a divergence that trips the check.

Skip it only where there is nothing to start: a milestone that touched documentation, configuration or rules and produced no runnable change.

Done: verify's report is in hand with every channel carrying a value, or the milestone is confirmed to have produced nothing runnable.

## Step 10 — Review

Run the reviewer agents named in Step 2: `code-reviewer` and each stack reviewer, plus `security-reviewer` if flagged. Invoking this skill is the request for those agents — dispatch them without re-asking. Scope every one of them to this milestone's diff, never the whole codebase: the codebase-wide audit is a separate gate that runs once, after the last milestone. Reviewing the diff yourself is not a substitute and does not partly satisfy this gate; if an agent cannot run, the gate is not passed — say so and stop for a decision. Resolve every CRITICAL and HIGH finding before the gate; record any accepted MEDIUM or LOW item.

If Step 2 flagged UI, also run a design review: screenshot the built screen (via the `run`
skill where the environment supports it) and check it for conformance to `docs/DESIGN.md`,
plus the `design-quality.md` (aesthetics) and `design-principles.md` (craft floor) rules —
verified visually, not asserted. For a backend-only milestone this review does not run.

The design review checks that what shipped matches the decisions Step 5 recorded — including that
the keyboard paths signed off there actually work. It does not re-open those decisions: an
interaction decision carries no severity, and "this surface has no keyboard shortcut" is not a
finding when Step 5 recorded that answer. An interaction decision left *unanswered* is a gap in
Step 5, to be resolved there, not escalated here.

Before dispatching, snapshot the tree — this step runs before the commit gate, so the work under review is its own only copy. Run `git stash create` and record the commit id it prints; it writes a dangling commit and does **not** touch the working tree. Copy any untracked files `git status --porcelain` marks `??` into `.claude/scratch/`, since the stash commit does not capture them. Record `git diff --stat` and `git status --porcelain` as the before-figures, and re-run both when the agents return: compare before acting on any finding, per `agents.md`. If they diverge, stop and tell the user, naming the snapshot commit id and the scratch copies.

**Then review the fix**, per `code-review.md`'s "The fix gets reviewed too": one further `code-reviewer` dispatch over the fix diff only. A new CRITICAL or HIGH from that pass is a design signal, not a cue to fix again — stop and raise it.

Done: reviews are clean, or their CRITICAL and HIGH findings are fixed, re-verified, and the fix diff has been through one further review pass — and the tree matches its before-figures.

## Step 11 — Stop at the commit gate

Do not commit or merge — that ritual belongs to `wrap-it-up`. Confirm the milestone against its done-criteria, then recommend the next move:

- **Done-criteria met** → recommend `/wrap-it-up` to reconcile the docs, perform the git ritual, and open the next phase.
- **Work remains, or this is mid-task** → recommend `/save-session` to capture the handoff.

Done: the milestone is reported as complete or partial against its done-criteria, and the matching closer is recommended.
