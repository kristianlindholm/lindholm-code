---
name: implement-task
description: Execute one parked task from the project checklist to best practice, stopping at the commit gate.
disable-model-invocation: true
---

# Implement Task

Carries one parked task from `docs/CHECKLIST.md` to review-passed, working code, applying only the practices that fit the work.

Stop at the commit gate: do not commit or merge, and do not delete the task line. Do not write implementation before Step 3 has scoped what applies.

The skill has two legitimate endings: the task built and awaiting commit, or the task closed at Step 2 because its target is already met. Building is not the only correct outcome.

This skill is the single-task sibling of `implement-milestone`. Both sequence the pipeline defined in `development-workflow.md`; that rule, with `agents.md`, `testing.md`, and `code-review.md`, stays the source of truth for each stage. Point to them, do not relearn them here. A single task is smaller than a milestone, so the heavyweight stages (`planner`, `architect`, ADRs) usually do not apply — Step 3 decides.

## Step 1 — Load the task

Read `docs/CHECKLIST.md`. Select the open `[ ]` item by the number given on the command line (counting open items top to bottom). If no number was given, list the open items numbered and close with a single `Which? (1-N)`, then wait (global interaction-design doctrine). Ignore `[x]` items — those are done and awaiting commit.

The entry has no formal done-criteria the way a milestone does. Read the entry text and its `while:` context, then state a concrete done-criterion for the task — what must be true for it to count as finished. Read `CLAUDE.md` and the PRD (`docs/PRD.md`) for constraints that bind the work; product code lives under `product/` per `CLAUDE.md`'s Project Layout.

Done: one open task is selected, and you can state its scope, a concrete done-criterion, and the part of the codebase it touches.

## Step 2 — Test the target against the product

Step 1 wrote a target. Measure it against the code as it stands before scoping or building anything.

A parked task is written at one time and executed at another, with arbitrary work landing in between. `save-for-later` records only the task text, a date, and a one-clause `while:` — by design, so that parking never interrupts — so the entry carries no premise to check itself against. The check has to happen here, and it has to happen for every item regardless of which skill parked it: work that shipped last week can make a `give-feedback` entry redundant exactly as easily as a `save-for-later` one.

Read the product code, not the project's history. The code is what the target is true or false against; commit subjects describe intent and can be close enough to the task's wording to mislead either way. Answer two questions:

- **Is this the real problem?** Confirm from the code that the entry describes what is actually wrong. The reported symptom may have a different root cause, which reframes the item. This is the investigation `give-feedback` Step 1 performs before parking; for an item parked by `save-for-later`, nobody has performed it.
- **Is the target already met, and how much of it?** Not whether anything adjacent shipped — whether the stated done-criterion is true now, and if only partly, which specific part is missing.

Then scan the other open `[ ]` entries for the same work. Two items can describe one job before either is built, and no amount of code reading finds that.

| Conclusion | Disposition |
|---|---|
| Target not met, entry accurate | Continue to Step 3 |
| Same work as another open item | Merge the targets, state the combined done-criterion, continue to Step 3 |
| Entry describes the wrong problem | State the reframed done-criterion, continue to Step 3 |
| Target already met, fully or substantially | Stop and offer to close (below) |

Only the last conclusion means do not build. The other three adjust the target and carry on.

### When the target is already met

Stop and offer to close. Never close an item on your own initiative — whether work still has value is the user's call, not yours.

**The default at this stop is to close.** The test: state what is left as its own done-criterion, then ask whether you would park that as a fresh item today. If not, the item closes.

The pull runs the other way, so name it. Surviving gaps are nameable and the delivered part is diffuse — "these four cases still fail" writes itself, "the other ninety percent already works" does not — so listing the gaps makes them read as the whole item. A remnant always exists. Recommending the build because you found one is the failure this step exists to prevent.

Present the measurement, then numbered options with the recommendation marked, per the global interaction-design doctrine.

Worked example:
> A control shipped last week already produces the parked item's behaviour, except for four edge cases it does not cover.
> Right: "The target is met by <control>, except <the four gaps>. Stated as its own item, that gap coverage is not something you would park today. Recommend closing. 1. Close it 2. Build the gap coverage only 3. Build as written. Which? (1-3)"
> Wrong: "The premise is partly already shipped — here are two ways to build it." (recommends building because the gaps are the nameable part)

On a decision to close, go to Step 7: the flip and the commit gate are identical for a resolved item and a built one. On a decision to build a narrowed version, make that the target and continue to Step 3.

Done: the target is confirmed not-yet-met and stated in final form — adjusted for any same-work sibling and any reframing the code revealed — or the skill has stopped and offered to close because the target is already met.

## Step 3 — Scope what applies

Map the task to an approach, the same gate `implement-milestone` uses, sized for one task:

- **Test-first (`tdd` skill)** — applies wherever the task adds or changes behavior. Skip only where there is nothing to assert: docs, config, a trivial rename.
- **Reviewer agents** — the general `code-reviewer`, plus the stack's reviewer for each language touched (`go-reviewer`, `react-reviewer`, ...).
- **`security-reviewer`** — only when the work hits one of `code-review.md`'s security triggers (auth, user input, data access, secrets, crypto).
- **`planner` / `architect`** — normally not needed for a single task. Invoke `planner` only if the task turns out to span components; record an ADR only if a genuine hard-to-reverse decision arises.

Done: the task is mapped to test-first or direct implementation, and the agents that will run — and at which stage — are named.

## Step 4 — Research before building

Before writing new code, run the Research & Reuse step in `development-workflow.md`: search for an existing implementation, library, or pattern and prefer it over hand-rolled code.

Done: an existing approach is adopted, or none fits and that is confirmed.

## Step 5 — Build red-green

For every part Step 3 marked test-first, run the `tdd` skill: a failing test (red), the minimal code to pass (green), then refactor. Implement the rest directly against the done-criterion. Write product code under `product/` and run build/test commands from there. Hand a failing build to the stack's `*-build-resolver`: invoking this skill is the request for that agent, so dispatch it rather than quietly fixing the build yourself.

Done: the task's done-criterion is met and the test and build commands pass — confirmed with evidence, not assumed.

## Step 6 — Review

Run the reviewer agents named in Step 3: `code-reviewer` and each stack reviewer, plus `security-reviewer` if flagged. Invoking this skill is the request for those agents — dispatch them without re-asking. Scope every one of them to this task's diff, never the whole codebase: the codebase-wide audit is a separate gate that runs once, after the last milestone. Reviewing the diff yourself is not a substitute and does not partly satisfy this gate; if an agent cannot run, the gate is not passed — say so and stop for a decision. Resolve every CRITICAL and HIGH finding before the gate; record any accepted MEDIUM or LOW item.

Done: reviews are clean, or their CRITICAL and HIGH findings are fixed and re-verified.

## Step 7 — Mark done, stop at the commit gate

Confirm the work against the done-criterion as it stands after Step 2. On verified completion, flip that entry in `docs/CHECKLIST.md` from `- [ ]` to `- [x]`, leaving the line and its `while:` context otherwise unchanged. Do not delete the line. Do not commit or merge.

Flip **every** entry the work satisfied, not only the one that was invoked. If Step 2 merged a same-work sibling, that sibling is flipped too — left open, it survives as a task for work already done and the next session investigates it from scratch.

An item closed at Step 2 arrives here too, and takes the same path: it is resolved, so it flips to `- [x]` and `wrap-it-up` commits and removes it exactly as it would a built one. Report that it closed without code and state the measurement behind it, so the commit message carries why.

Report the task as done and awaiting commit, and tell the user that `/wrap-it-up` will commit it and remove the line. Never invoke the milestone close from here — a single task does not close a milestone. If a milestone is currently in-progress, `/wrap-it-up` commits the task on its own (its boundary guard will not redirect to `/save-session` for a task commit); if a milestone is also completing, the task commit rides along with that wrap.

Done: the task is verified against its done-criterion — or resolved at Step 2 — every entry it satisfied reads `- [x]`, nothing is committed, and the user is pointed at `/wrap-it-up`.
