---
name: implement-task
description: Execute one parked task from the project checklist to best practice, stopping at the commit gate.
disable-model-invocation: true
---

# Implement Task

Carries one parked entry from `docs/CHECKLIST.md` to review-passed, working code, applying only the practices that fit the work.

An entry is usually one task. It may also be a **bundle** — one `[ ]` line holding several plain-bullet pieces that `give-feedback` grouped because they touch the same files or surface. A bundle is still one entry, one selection, one review and one commit; what multiplies is the number of targets to state and meet, not the number of passes through this skill. Where a step below says "the task", read it as "each piece of the entry" when the entry is a bundle.

Stop at the commit gate: do not commit or merge, and do not delete the task line. Do not write implementation before Step 3 has scoped what applies.

The skill has two legitimate endings: the task built and awaiting commit, or the task closed at Step 2 because its target is already met. Building is not the only correct outcome.

This skill is the single-task sibling of `implement-milestone`. Both sequence the pipeline defined in `development-workflow.md`; that rule, with `agents.md`, `testing.md`, and `code-review.md`, stays the source of truth for each stage. Point to them, do not relearn them here. A single task is smaller than a milestone, so the heavyweight stages (`planner`, `architect`, ADRs) usually do not apply — Step 3 decides.

## Step 1 — Load the task

Read `docs/CHECKLIST.md`. Select the open `[ ]` item by the number given on the command line (counting open items top to bottom). If no number was given, list the open items numbered and close with a single `Which? (1-N)`, then wait (global interaction-design doctrine). Ignore `[x]` items — those are done and awaiting commit.

**Count `[ ]` lines only.** A bundled entry's pieces are plain bullets and are not numbered; the whole bundle is one number. If you find a checkbox on an indented sub-item, that is a malformed entry — say so and treat it as part of its parent rather than as an item of its own, or every number after it is wrong.

**One entry, not several.** Selecting a second entry because the first looked small is how a session ends up with one commit carrying five unrelated concerns and one reviewer dispatch covering all of it. If the user asks for several — "do 2 and 3", "as many as possible" — take them, but say which ones and keep each entry's work as its own commit at Step 9. The grouping that belongs together was already decided when the entry was written; do not re-form it here.

The entry has no formal done-criteria the way a milestone does. Read the entry text and its `while:` context, then state a concrete done-criterion — what must be true for it to count as finished. **A bundled entry gets one done-criterion per piece**, each stated separately in that piece's own terms; the entry is finished when all of them are met. Read `CLAUDE.md` and the PRD (`docs/PRD.md`) for constraints that bind the work; product code lives under `product/` per `CLAUDE.md`'s Project Layout.

`CLAUDE.md` is an index — the rules binding every change, plus a `## Rules` table pointing at the topic files holding the rest. **Do not read those files yet, and do not read them all**: Step 3 scopes the task and decides which ones it needs.

Done: one open entry is selected, and you can state its scope, the part of the codebase it touches, and a concrete done-criterion — one per piece where the entry is a bundle.

## Step 2 — Test the target against the product

Step 1 wrote a target. Measure it against the code as it stands before scoping or building anything. **A bundled entry is measured piece by piece** — one pass of everything below per target, with its own conclusion. A bundle was grouped by the files its fixes touch, which says nothing about whether any one of them is still needed.

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

**Where a bundle's pieces reach different conclusions**, each piece takes its own row. A piece whose target is already met is dropped from the build with its measurement stated, and the entry carries on with the rest — one met piece does not close the entry, and it does not earn its own stop. Only when **every** piece is already met does the entry take the "already met" path below. Record what was dropped and why; Step 9 writes it into the report so the commit message carries it.

### When the target is already met

Stop and offer to close. Never close an item on your own initiative — whether work still has value is the user's call, not yours.

**The default at this stop is to close.** The test: state what is left as its own done-criterion, then ask whether you would park that as a fresh item today. If not, the item closes.

The pull runs the other way, so name it. Surviving gaps are nameable and the delivered part is diffuse — "these four cases still fail" writes itself, "the other ninety percent already works" does not — so listing the gaps makes them read as the whole item. A remnant always exists. Recommending the build because you found one is the failure this step exists to prevent.

Present the measurement, then numbered options with the recommendation marked, per the global interaction-design doctrine.

Worked example:
> A control shipped last week already produces the parked item's behaviour, except for four edge cases it does not cover.
> Right: "The target is met by <control>, except <the four gaps>. Stated as its own item, that gap coverage is not something you would park today. Recommend closing. 1. Close it 2. Build the gap coverage only 3. Build as written. Which? (1-3)"
> Wrong: "The premise is partly already shipped — here are two ways to build it." (recommends building because the gaps are the nameable part)

On a decision to close, go to Step 9: the flip and the commit gate are identical for a resolved item and a built one. On a decision to build a narrowed version, make that the target and continue to Step 3.

Done: every target the entry carries is confirmed not-yet-met and stated in final form — adjusted for any same-work sibling and any reframing the code revealed — with any already-met piece dropped and its measurement recorded, or the skill has stopped and offered to close because the whole entry is already met.

## Step 3 — Scope what applies

Map the task to an approach, the same gate `implement-milestone` uses, sized for one task:

- **Test-first (`tdd` skill)** — applies wherever the task adds or changes behavior. Skip only where there is nothing to assert: docs, config, a trivial rename.
- **Reviewer agents** — the general `code-reviewer`, plus the stack's reviewer for each language touched (`go-reviewer`, `react-reviewer`, ...).
- **`security-reviewer`** — only when the work hits one of `code-review.md`'s security triggers (auth, user input, data access, secrets, crypto).
- **Build-resolver** — the stack's `*-build-resolver` for each language touched, for Step 6. Name it here even though it may not be needed, so Step 6 has a list rather than a search. Not every language has one and not every project provisions one: if `.claude/agents/` holds no resolver for this stack, record that here and Step 6 clears the build without a dispatch.
- **`planner` / `architect`** — normally not needed for a single task. Invoke `planner` only if the task turns out to span components; record an ADR only if a genuine hard-to-reverse decision arises. Answer the ADR question here either way, in writing — a bundled entry puts several pieces of work through this one scoping pass, so it is the only place the question gets asked for any of them.
- **UI** — does any piece of this entry change what a user sees or how a surface behaves? Answer yes or no in writing, and on yes name the surfaces. Step 7 runs on this answer, so an unanswered question is a skipped design pass. Spacing, labels, wording on a control, an overlap, a focus order, a new readout and a new interaction all count; a pure rename behind the surface, an escaping fix and a doc change do not.

**Then name the rules this scope selects, in writing, before opening any of them** — by `implement-milestone` Step 2's selection list, which this step follows exactly: read `CLAUDE.md`'s `## Rules` index and `docs/adr/README.md`, write the list with what selected each entry, then open exactly that list. A single task usually selects one or two.

**Do not list, glob or search `docs/rules/`, `docs/architecture/` or `docs/adr/`.** The index is how you find these files; sweeping the directories reads all of them and turns selection into something you describe afterwards rather than decide first. A file not in the index is not yours to open; one the index names that does not exist is reported and carried on without. Where an older ADR index gives only a number, opening `docs/adr/<NNNN>-*.md` by that number is allowed — the slug cannot be derived from the title.

Done: the task is mapped to test-first or direct implementation, the agents that will run — and at which stage — are named, the UI question is answered yes or no with its surfaces named on yes, any hard-to-reverse decision the entry surfaced has an ADR or none arose and that is stated — the negative recorded for the same reason `implement-milestone` Step 4 records it, so the question was asked rather than merely unanswered — and **the selected list was written before anything was opened, what you opened matches it exactly, no rules directory was swept, and you can name what you deliberately skipped.**

## Step 4 — Research before building

Before writing new code, run the Research & Reuse step in `development-workflow.md`: search for an existing implementation, library, or pattern and prefer it over hand-rolled code.

Done: an existing approach is adopted, or none fits and that is confirmed.

## Step 5 — Build red-green

For every part Step 3 marked test-first, run the `tdd` skill: a failing test (red), the minimal code to pass (green), then refactor. Implement the rest directly against the done-criterion. Write product code under `product/` and run build/test commands from there.

Compile and type errors met along the way are part of the cycle; fix them and carry on. Do not dispatch an agent mid-cycle — Step 6 is where a build that will not come green is handed off.

Done: the task's done-criterion is met, and each part's covering tests have been run — whatever they report.

The covering tests, not the whole suite. This is the inner loop, where the proving command is the narrowest one that settles the claim. Step 6 runs the full build and suite at the point the dispatch decision is made; running them here as well measures the same tree twice and buys nothing.

## Step 6 — Clear the build

Run the build, typecheck and test commands now, and read the output.

**Green — say so and go on.** Nothing to dispatch.

**Red — dispatch the build-resolver named in Step 3 over the failures.** Invoking this skill is the request for that agent; dispatch it rather than fixing the build yourself. Give it the failing command and its output, scoped to this task's work. When it returns, re-run the commands yourself: its report that the build is fixed is not evidence, the command is.

**Red, and Step 3 recorded no resolver for this stack** — fix it yourself and say so. Most projects have no build-resolver provisioned, and several supported languages have none in the store at all; that is a coverage decision, not a reason to stall.

Clearing the build is its own step, not a clause inside Step 5. Do not fold it back in: a dispatch asked for mid-cycle does not happen, and one you arrive at does.

If the resolver cannot fix it, stop and report.

Done: the build, typecheck and test commands pass — confirmed by running them after any dispatch, not by an agent's report.

## Step 7 — Look at it (only if Step 3 answered yes to UI)

Step 3 answered the UI question and named the surfaces. On a no, say so in one line and go to Step 8. On a yes, this runs — **once for the whole entry**, over every surface it named, not once per piece.

Launch the app (the `run` skill) and screenshot each of those surfaces. Then look at the images. Reading the markup back is not this step: a label that overlaps an axis, spacing that collapses, a control that covers the thing it describes, a type scale that flattens at the rendered size — none of them are visible in the CSS that produced them, and a geometry assertion in a fake DOM proves only the number you thought to assert. The defect that put the entry on the checklist was almost always spotted by eye; it is closed by eye too.

Check each image against:

- **`docs/DESIGN.md`** — the project's signed-off visual language. The work composes its tokens and conventions; it does not introduce new ones. A piece that genuinely overturns a signed-off row marks that row REVISED with its `*Was:*` text rather than editing it away quietly.
- **`design-quality.md` and `design-principles.md`**, where the project provisions them — the anti-template ceiling and the craft floor: legibility, focus states, spacing, the declared viewport range.

Fix what the images show, then look again. Where the environment cannot screenshot, say so plainly and run the pass against the rendered page by whatever means it does offer; do not pass over it in silence.

**This is its own step, before Step 8, for two reasons.** A critique asked for while building does not happen and one you arrive at does — the same reason Step 6 is not a clause inside Step 5. And starting the app writes build artifacts into the working tree, while Step 8 opens by snapshotting it and comparing before-figures; looking first makes those writes part of the baseline instead of a divergence no agent appears to have caused.

Done: every surface Step 3 named has been seen as an image and checked against `docs/DESIGN.md` and the project's design rules, with what that turned up either fixed or written down — or Step 3 answered no and that is stated.

## Step 8 — Review

Run the reviewer agents named in Step 3: `code-reviewer` and each stack reviewer, plus `security-reviewer` if flagged. Invoking this skill is the request for those agents — dispatch them without re-asking. Scope every one of them to this task's diff, never the whole codebase: the codebase-wide audit is a separate gate that runs once, after the last milestone. Reviewing the diff yourself is not a substitute and does not partly satisfy this gate; if an agent cannot run, the gate is not passed — say so and stop for a decision. Resolve every CRITICAL and HIGH finding before the gate; record any accepted MEDIUM or LOW item.

Before dispatching, snapshot the tree — this step runs before the commit gate, so the work under review is its own only copy. Run `git stash create` and record the commit id it prints; it writes a dangling commit and does **not** touch the working tree. Copy any untracked files `git status --porcelain` marks `??` into `.claude/scratch/`, since the stash commit does not capture them. Record `git diff --stat` and `git status --porcelain` as the before-figures, and re-run both when the agents return: compare before acting on any finding, per `agents.md`. If they diverge, stop and tell the user, naming the snapshot commit id and the scratch copies.

**Then review the fix**, per `code-review.md`'s "The fix gets reviewed too": one further `code-reviewer` dispatch over the fix diff only. A new CRITICAL or HIGH from that pass is a design signal, not a cue to fix again — stop and raise it.

Done: reviews are clean, or their CRITICAL and HIGH findings are fixed, re-verified, and the fix diff has been through one further review pass — and the tree matches its before-figures.

## Step 9 — Mark done, stop at the commit gate

Confirm the work against the done-criteria as they stand after Step 2 — every one of them where the entry is a bundle. On verified completion, flip that entry in `docs/CHECKLIST.md` from `- [ ]` to `- [x]`, leaving the line, its sub-bullets and its `while:` context otherwise unchanged. Do not delete the line. Do not commit or merge.

**A bundled entry flips whole or not at all.** There is one checkbox and it is on the parent line, so there is no half-flip to write: ticking a sub-bullet is malformed (`wrap-it-up` deletes every `[x]` line it finds, and would take that bullet out of an entry still open). If a piece cannot be finished — blocked, reframed into something larger, or deferred by the user — take it out of the bundle and append it as its own new `[ ]` entry carrying its own text and `while:` line, then flip the parent. The entry then means what it says: everything left inside it is done.

Flip **every** entry the work satisfied, not only the one that was invoked. If Step 2 merged a same-work sibling, that sibling is flipped too — left open, it survives as a task for work already done and the next session investigates it from scratch.

An item closed at Step 2 arrives here too, and takes the same path: it is resolved, so it flips to `- [x]` and `wrap-it-up` commits and removes it exactly as it would a built one. Report that it closed without code and state the measurement behind it, so the commit message carries why. The same goes for a bundle's individual piece that Step 2 dropped as already met: name it in the report with its measurement, since it leaves no trace in the diff.

Report the task as done and awaiting commit, and tell the user that `/wrap-it-up` will commit it and remove the line. **One commit's worth of work per entry.** Where the user asked for several entries in one go, each entry is still its own commit — `wrap-it-up` derives each commit message from that entry's own text, and one commit spanning several entries buries what each was for and hands the reviewer a diff covering concerns with nothing to do with each other. Never invoke the milestone close from here — a single task does not close a milestone. If a milestone is currently in-progress, `/wrap-it-up` commits the task on its own (its boundary guard will not redirect to `/save-session` for a task commit); if a milestone is also completing, the task commit rides along with that wrap.

Done: every done-criterion the entry carries is verified — or resolved at Step 2 — every entry the work satisfied reads `- [x]` with no sub-bullet ticked, any unfinished piece is re-parked as its own `[ ]` entry, nothing is committed, and the user is pointed at `/wrap-it-up`.
