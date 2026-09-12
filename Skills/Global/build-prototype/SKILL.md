---
name: build-prototype
description: Build a prototype project's whole milestone plan in one continuous run - no stop between milestones, no review gate - recording what was built so it can be promoted later.
disable-model-invocation: true
---

# Build Prototype

Carries a prototype from its first milestone to its last in one run. Where `implement-milestone`
builds a single milestone and stops at the commit gate, this builds the entire plan without
stopping, and records what it built so `promote-prototype` has a contract to preserve.

**Prototype mode relaxes engineering rigor, not product thinking.** Requirements, the design
direction, the milestone plan, and the product map were all settled in full at `new-project`.
What this skill drops is the ceremony around the code: the confirmation gate between milestones,
the coverage target, and the reviewer agents. Nothing here licenses building the wrong thing
quickly.

This skill sequences the same pipeline as `implement-milestone` and points at the shared parts
rather than restating them. Where a step below says "as `implement-milestone` Step N does", that
skill is the source of truth and this one must not drift from it.

Only for a project whose `.claude/wrap-it-up.json` records `"mode": "prototype"`.

## Step 1 — Confirm the mode and load the plan

Read `.claude/wrap-it-up.json`. If `mode` is absent or is not `prototype`, stop and say so: this
is a product project, and `/implement-milestone` is the skill for it. Do not flip the flag — the
mode was chosen at `new-project` Gate 0, and changing it is not this skill's decision.

Then load the plan as `implement-milestone` Step 1 does: every milestone's scope and
done-criteria from the Delivery Milestones table in `docs/PROGRESS.md`, the `RESUME HERE` pointer
if present, and `CLAUDE.md` plus `docs/PRD.md` for the constraints that bind the work. Product
code lives under `product/`.

A prototype has no version control (Gate 6), so there is no baseline diff to read and nothing to
roll back to. The working tree is the only copy of everything this skill produces.

Done: the mode is confirmed as prototype, and every milestone in the plan is loaded in order with
its scope and done-criteria.

## Step 2 — Run the plan without stopping

Work the milestones in their `docs/PROGRESS.md` order, start to finish, in one run. Do not ask for
confirmation between them, and do not stop at a commit gate — there is nothing to commit.

For each milestone:

- **Research** — as `implement-milestone` Step 3 does. Reuse beats hand-rolling, and it is faster,
  which is the whole point here.
- **UI surfaces** — derive them from `docs/DESIGN.md`, which Gate 3 established in a single pass.
  Do not reinvent a visual language per screen. Resolve the interaction decisions a surface needs
  yourself and record the answers in `docs/DESIGN.md`: with no stops there is nobody to ask, and an
  unrecorded decision is one the next milestone re-improvises.
- **Build** — write the code to the done-criteria. Product code, tests, and the manifest all live
  under `product/`.
- **Tests** — at your judgment, case by case. Test the logic that is easy to get wrong; skip the
  wiring. There is no coverage target and no test-first requirement. Judgment means an actual
  decision per milestone and not a standing "skip" — a milestone containing a calculation, a
  parser, or a state machine gets a test.
- **Build failures** — hand to the stack's `*-build-resolver`. Invoking this skill is the request
  for that agent; dispatch it rather than fixing the build by hand.

No `code-reviewer`, no stack reviewer, no `security-reviewer`, and no design review. All of them
are deferred to `promote-prototype`. Do not run them here, and do not substitute a review of your
own — the point of a prototype is to find out whether the product works, and reviewing it is what
promotion is for.

Then run Step 3 for that milestone, mark its row complete in `docs/PROGRESS.md`, and go straight
into the next milestone.

Done: every milestone's done-criteria are met and the app builds and runs — confirmed by running
it, not assumed.

## Step 3 — Record and re-check at every milestone boundary

This runs after each milestone, before the next one starts. It is what makes the prototype
promotable, and it is the only thing standing in for the confirmation gate that was removed.

1. **Append to `docs/FEATURES.md`** — one line per feature this milestone added, written from the
   user's side: what they can now do, and what they get back. Create the file on the first
   milestone, with this shape:

       # FEATURES — [Project Name]

       > What this prototype does, recorded as it was built. This is the contract that
       > promote-prototype preserves: the implementation may be rewritten, these behaviours
       > may not change.

       ## Milestone 1 — [name]
       - [what a user can do, and what they get back]

   Write it while you still know whether a behaviour was deliberate. Reconstructed later from the
   code, an accident is indistinguishable from a decision.

2. **Capture the new screens** — launch the app (the `run` skill) and screenshot every surface this
   milestone added, into `docs/captures/`. Name each file for the surface rather than the
   milestone, so the next capture overwrites it in place.

3. **Re-capture every earlier screen** — relaunch and re-capture every surface captured so far.
   This is the regression check: a milestone that broke an earlier screen shows up here as a blank
   page, an error, or a layout that collapsed.

4. **Report breakage immediately.** A screen that broke is reported in one line and fixed before
   the next milestone starts. Never carry known breakage forward — the run has no other
   checkpoint, so a break left here surfaces at the end with no way to tell when it happened.

Where the app cannot be captured — a native desktop or mobile build, or a CLI with no screen — say
so plainly, once, keep the feature log alone, and substitute what the environment allows: relaunch,
run the main command, and check that the earlier commands still work.

Done: `docs/FEATURES.md` covers every milestone built so far, `docs/captures/` holds a current
screenshot of every surface, and nothing captured earlier is broken.

## Step 4 — Stop only for a genuine fork

Stop mid-run only where guessing wrong costs more than asking: a decision `docs/PRD.md`, the design
foundation, and the product map all leave open, where the two paths lead somewhere materially
different and the wrong one would have to be unpicked.

Not a reason to stop: a naming choice, a library choice, a layout detail, an interaction decision
Step 2 tells you to resolve yourself, or wanting confirmation that things are going well. Those are
exactly what "no stops" removed.

When you do stop, ask one question following the global interaction-design doctrine, then resume
the run from where it paused.

Done: the run either completed uninterrupted, or paused on a named fork, got an answer, and
continued.

## Step 5 — Closing

Report against the plan: which milestones were built, what `docs/FEATURES.md` now records, and
anything left unfinished. State plainly what was not done — no reviews, no coverage target, no
version control — so the prototype's state is never overstated.

Then run `/wrap-it-up`. A prototype is `gitBackend: none`, so it reconciles the documents and runs
no git ritual — and it is the only thing in the store that writes the `RESUME HERE` pointer into
`docs/PROGRESS.md`. Without that pointer, `continue-project` reads a finished prototype as a first
session and never offers to promote it. Do not write the pointer here instead: `wrap-it-up` owns
`docs/PROGRESS.md`, and a second writer is how the two drift apart.

Then recommend the next move:

- **Every milestone complete** → recommend `/promote-prototype` to publish it and begin hardening.
- **Work remains** → recommend `/save-session` to capture the handoff, noting that with no version
  control the working tree is the only copy.

Done: the run is reported against the plan, what was not done is stated plainly, the documents are
reconciled, and the matching closer is recommended.
