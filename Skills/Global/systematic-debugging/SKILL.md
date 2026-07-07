---
name: systematic-debugging
description: Systematic debugging. Use when encountering any bug, test failure, build failure, or unexpected behavior, before proposing or attempting a fix.
---

# Systematic Debugging

## Core principle

Find the root cause before attempting any fix. A fix aimed at a symptom is a failure even when it makes the symptom disappear — it leaves the cause in place and usually creates a new bug elsewhere.

**The iron law:** no fix without root-cause investigation first. If Phase 1 is not complete, you cannot propose a fix.

This applies most when it feels least necessary: under time pressure, when "one quick fix" looks obvious, when a previous fix did not work, or when the issue looks too simple to investigate. Simple bugs have root causes too, and the process is fast for simple bugs.

## The four phases

Complete each phase before starting the next.

### Phase 1 — Root-cause investigation

Before attempting any fix:

- **Read the error completely.** Read the full message and stack trace, including line numbers, file paths, and error codes. The exact cause is often stated in text already skimmed past.
- **Reproduce it consistently.** Establish the exact steps and whether it happens every time. If it is not reproducible, gather more data rather than guessing.
- **Check recent changes.** What changed that could cause this — a diff, recent commits, a new dependency, a config or environment difference?
- **Instrument component boundaries** (multi-component systems). When the system spans layers (CI to build to signing, API to service to database), add diagnostic logging at each boundary: what data enters, what data exits, whether config and environment propagate. Run once to see _where_ it breaks, then investigate that component. See [root-cause-tracing.md](root-cause-tracing.md) for tracing a bad value backward through the call chain to its origin.

### Phase 2 — Pattern analysis

- **Find a working example.** Locate similar code in the same codebase that works. What is different between the working path and the broken one? List every difference; do not dismiss any as "that can't matter."
- **Read the reference completely.** When following a pattern or reference implementation, read all of it, not a skim. Partial understanding guarantees subtle bugs.
- **Understand the dependencies.** What components, settings, environment, or assumptions does the broken code rely on?

### Phase 3 — Hypothesis and minimal test

- **State one hypothesis explicitly:** "The root cause is X because Y." Be specific.
- **Test it with the smallest possible change**, one variable at a time. Do not change several things at once — you will not know which mattered, and combined changes cause new bugs.
- **Confirm or replace.** If the change confirms the hypothesis, move to Phase 4. If not, form a new hypothesis — do not stack another fix on top.
- **When you do not know, say so.** State what is not understood and investigate or ask, rather than pretending certainty.

### Phase 4 — Fix at the source

- **Write a failing test first.** Reproduce the bug with the simplest possible automated test. Use the [`tdd`](../tdd/SKILL.md) skill — the test must fail for the intended reason before the fix, and pass after. A bug fixed without a test does not stay fixed.
- **Make one fix, at the root cause.** No bundled refactoring, no "while I'm here" improvements.
- **Verify with evidence.** The new test passes, no other test broke, and the original symptom is gone — confirmed by running the commands, not assumed.
- **Consider defense in depth.** Once the root cause is fixed, add validation at the layers the bad value passed through so the bug becomes structurally impossible. See [defense-in-depth.md](defense-in-depth.md).

## When fixes keep failing

Count the attempts. After three failed fixes, stop fixing and question the design.

The pattern that signals a design problem: each fix reveals a new problem in a different place, each fix requires increasingly large refactoring, or each fix creates a new symptom elsewhere. That is not a failed hypothesis — it is a wrong architecture. Do not attempt fix number four. Raise the structural question with the user: is this pattern sound, or is it being kept through inertia?

## Rationalization table

| Excuse | Reality |
|--------|---------|
| "The issue is simple, no need for the process." | Simple issues have root causes too. The process is fast for them. |
| "Emergency, no time for the process." | Systematic debugging is faster than guess-and-check thrashing. |
| "Just try this one change first, then investigate." | The first fix sets the pattern. Do it right from the start. |
| "I'll write the test after I confirm the fix works." | Untested fixes do not stick. The failing test first is what proves the fix. |
| "Multiple changes at once will save time." | You cannot isolate what worked, and it introduces new bugs. |
| "The reference is long; I'll adapt the pattern from a skim." | Partial understanding guarantees bugs. Read it completely. |
| "I can see the problem, let me fix it." | Seeing the symptom is not understanding the root cause. |
| "One more fix attempt." (after 2+ failures) | Three failures means a design problem. Question the pattern, do not fix again. |

## Red flags — stop and return to Phase 1

- "Quick fix now, investigate later."
- "Just change X and see if it works."
- Proposing solutions before tracing the data flow.
- "I don't fully understand it, but this might work."
- Listing fixes before any investigation.
- "One more fix attempt" when two or more have already failed.
- Each fix revealing a new problem in a different place.

## When investigation finds no root cause

If thorough investigation shows the issue is genuinely environmental, timing-dependent, or external: document what was investigated, implement appropriate handling (retry, timeout, clear error message), and add logging for future diagnosis. But most "no root cause" conclusions are incomplete investigation — be sure Phase 1 was actually finished.

## Supporting techniques

- [root-cause-tracing.md](root-cause-tracing.md) — trace a bad value backward through the call chain to its original trigger.
- [defense-in-depth.md](defense-in-depth.md) — after fixing the root cause, validate at every layer so the bug cannot recur.
- [condition-based-waiting.md](condition-based-waiting.md) — replace arbitrary delays with condition polling to remove flaky-test race conditions.

## Related artifacts

- The [`tdd`](../tdd/SKILL.md) skill — Phase 4 writes the failing test through its RED-GREEN gates.
- The `code-reviewer` agent — request review after a non-trivial fix, per [`code-review.md`](../../../Rules/Global/code-review.md).
