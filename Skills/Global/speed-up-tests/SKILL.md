---
name: speed-up-tests
description: Use when a test suite is slow enough to be annoying — "the tests take forever", "why is the suite so slow", "can we run the tests in parallel", "should we switch to a faster test runner", or when a suite's runtime is growing faster than its test count. Finds where the time actually goes before changing anything.
---

# Speed Up Tests

Finds what a test suite's time is actually spent on, fixes the cost that dominates, and proves the
suite still observes the same things afterwards.

The standard this serves is one entry in the Global rule `testing.md`, "What a test costs to run":
a heavyweight resource starts once per run, never once per test. That rule prevents the debt. This
skill is for suites that already carry it.

**The iron law:** no change to how tests run until the time has been measured. Every slow suite has
an obvious suspect, and in the case that produced this skill the obvious suspect was innocent —
removing it made the suite slower.

## When to run this

The user is frustrated with suite time, a suite's runtime is outgrowing its test count, a run is
routinely started and then abandoned, or someone has proposed parallelism, a new runner, or
splitting across workers. Those three are answers; this finds the question.

## The one idea

**Find the cost a test pays regardless of what it asserts, and make it paid once per run instead of
once per test.**

That cost is always invisible in the test code, because test code only describes the assertion. It
shows up only when the time is measured. It is a browser process, a database container, a compile, a
dev server boot, a fixture rebuild, an auth handshake, a module graph rebuilt per file.

## Phase 0 — Protect the working tree

This work spawns processes, leaks temp directories, and patches the runner. None of that belongs in
the user's working tree, which may hold uncommitted work, and may be owned by another session.

- Copy the project (or just the test and source trees) into the scratchpad and work there.
- Record the prefix the suite's temp artifacts use, so a sweep can later target only those.
- Never commit an experiment. Nothing from this skill ships without a separate, deliberate change.

## Phase 1 — Baseline, written down before anything is touched

Run the full suite once, clean. Record total wall clock, test count, pass count, and what else was
running on the machine.

**Write the number down.** Every later comparison is against it, and a baseline recalled from memory
halfway through is how an unfair comparison gets made. If you measure a modified copy, baseline
*that copy too* — the same work measured 637 s in one tree and 995 s in another, and comparing
across them produces a confident, wrong conclusion.

## Phase 2 — Split the suite by what it costs

Classify every test by the heavy thing it touches: nothing (pure logic), a file, a database, a
server, a browser, a container. Then get per-file and per-test times.

Expect a sharp split. In the measured case 85% of tests were pure logic and finished in 16 seconds;
15% touched a browser and held 98% of the wall clock.

**State the split in one line before going further.** If one small group holds nearly all the time,
every optimisation aimed anywhere else is wasted — including the ones most often proposed.

## Phase 3 — Separate fixed cost from marginal cost

Two different numbers, and conflating them produces the wrong fix every time:

- **Fixed cost** — what one test pays to exist at all, before asserting anything.
- **Marginal cost** — what one more assertion costs inside a test already running.

Measure both:

1. **The bare resource.** Start the heavy thing, do the minimum, stop. No test, no harness.
2. **One real test.** The same thing through the harness, with one assertion.
3. **The same test with several more steps.** The difference is the marginal cost.

Measured: bare browser start-to-exit 4,519 ms; harness boot and one probe 5,322 ms; four additional
probes only +536 ms. So roughly 85% of every test was startup, and the marginal cost was near zero.

When the ratio is that lopsided, the unit of testing should be the **expensive resource**, not the
assertion — group related assertions into one session against one resource. This is not a licence to
write sprawling tests that assert eleven unrelated things. It is a statement that "one test per
behaviour" is a readability heuristic, not a law, and that paying a browser launch to assert one
boolean is a structural waste however tidy it reads.

**Name the shared resource out loud.** Everything after this depends on having identified it
correctly, and it is usually not what the suite's own code draws attention to.

## Phase 4 — Make the fixed cost run once

Reuse the resource and give each test an isolated slice — a fresh browser context, a rolled-back
transaction, a schema per worker, a namespaced key prefix.

Two constraints decide whether this is worth doing at all:

- **Test files must not change.** If the fix requires rewriting tests it is a migration, not a
  speed-up, and it puts the assertions at risk. Build the reuse behind the interface the tests
  already call. In the measured case all 45 files ran unmodified.
- **Isolation must survive.** Reuse the process, never the state. A test that passes only because an
  earlier test left something behind is worse than a slow test.

If a timing device is involved — a virtual clock, fake timers, a settle delay — carry it over
deliberately. Losing the virtual clock took one test from 13 s to 53 s while it still passed,
because the driver's waits silently became real.

## Phase 5 — Prove it, and diff the reports

Green is not the finish line. The suite now runs in a different environment, so:

1. Full suite in the modified copy. Record time and pass count.
2. Same copy, fix switched off. That is the fair baseline.
3. **Diff the two runs' output, not just the exit codes.** Every difference gets explained.

An unexplained difference is a finding — report it, do not absorb it. The measured case got 2.3x
faster and one test began reporting a scrolled element at a different position, because a virtual
clock now advanced at a different moment relative to the scroll. Every test was green. The suite
was, in one narrow respect, no longer observing the same thing.

Then sweep the temp artifacts the experiment created, by the prefix recorded in Phase 0. On this
project leaked harness directories had once filled the disk and broken seven unrelated tests, in a
way that looked exactly like a code defect.

## Phase 6 — Report, and let the user decide

In this order:

- Where the time goes (the Phase 2 split).
- What the fixed cost is (the Phase 3 numbers).
- What the fix achieves (before and after, measured in the same tree).
- What remains unexplained.
- What adoption costs — which files change, and what the risk is.

Adopting it is the user's call. The deliverable is a measurement and a working prototype, not a
merged change.

## Things that look like the answer and are not

Check each against the measurements before proposing it.

**Parallelism.** Only helps when the bottleneck is not the shared machine. Measure the knee before
adopting it and accept that it is sometimes 1. Measured across 492 test executions at three
concurrency settings: six-way was *slower* than one-at-a-time, and three-way on top of the real fix
went *red* with navigation timeouts. Never ship a concurrency change without a green run at that
setting — a faster suite that fails intermittently costs the time it saved plus the time spent
finding out whether the failure was real.

**A faster runner.** Fast at the part that was already fast. Check what the current runner does with
the logic tests before costing a migration: 899 of them ran in 16 seconds under a plain stdlib
runner, and no replacement wins more than a few of those. Where a modern runner genuinely is faster
for browser work it is because it drives a *persistent* browser underneath — which is Phase 4,
available without adopting anything. If you need a real browser from scratch, prefer a tool that
already solved process reuse over hand-rolling one; if a hand-rolled harness already exists and
works, fix its fixed cost rather than migrating.

**Splitting across subagents, workers, or tasks.** Same machine, same ceiling, plus a report to
verify instead of output to read. Across different machines it works and is the standard answer at
scale. The distinction is the shared resource, not the number of workers — so name the resource
before splitting, every time.

**Deleting tests for removed features.** Correct hygiene, negligible time. Do it, and prefer the
strong form: convert a removed feature's test into a *negative* test asserting the removal holds, so
the thing cannot quietly come back. But deleting twenty expensive tests saved a projected 72 seconds
where fixing the fixed cost saved five minutes. Prune for clarity, not for speed.

**Running fewer tests in the inner loop.** The one exception — this genuinely works, costs nothing,
needs no tooling, and should be proposed first. The covering file while working, the full suite at
the gate. The standard is already in the global `CLAUDE.md`: the proving command is the narrowest
one that settles the claim.

## The worked example

One project, measured 2026-11-25. Kept because the shape recurs, and because the numbers are what
make the phases above worth following rather than skipping.

| | |
|---|---|
| Suite | 1,063 tests, 45 files, vanilla JS, no build step |
| Usage | 192 full runs over 11 working days — mean 17.5/day, busiest 39 |
| Time spent waiting | ~35 hours |
| The split | 899 logic tests in 16 s; 164 browser tests holding 98% of wall clock |
| The suspect | a lock serialising browser tests — innocent; removing it made the suite *slower* |
| The actual cost | 4.7 s of Chrome startup wrapped around 0.8 s of work, paid 164 times |
| The fix | one persistent browser driven over CDP, a fresh context per test |
| Result | 995 s to 326 s, all 164 green, every test file unmodified |
| Left unexplained | one frozen-gutter position differing after a scroll, surviving a 250 ms settle |

The two lessons that did not come from the headline number: the obvious suspect was wrong, and the
suite got faster while quietly observing one thing differently.

## Related

- `testing.md` (Global rule) — "What a test costs to run" is the standard this serves; the rest of
  that file governs what to test, which this never relaxes.
- `code-review.md` (Global rule) — the checklist line that catches a per-test resource in the diff.
- `systematic-debugging` (Global skill) — the same discipline applied to defects. Reach for it if a
  speed-up makes a test fail and the reason is not immediately clear.
