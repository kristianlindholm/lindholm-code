---
name: verify
description: Use when the user asks whether an application actually runs right now — "is it still working", "does this still run", "check the app".
---

# Verify

Establishes whether an application actually runs, by starting it and reading what the running system
says — and reports every channel it read, including the quiet ones.

**Report nothing as a defect until it has survived a control.** A failure that reproduces on a case
the change never touched is the environment, not the application.

## What this is for, and what it is not

A test suite runs code with its boundaries stubbed. A reviewer reads a diff. Neither establishes
that the assembled application starts, that a dependency is reachable, or that a person can reach a
surface. That gap is the whole job.

> **Verify earns a check only where the test suite structurally cannot assert it.**

**In scope:** that the application starts as it actually runs; that each dependency answers; that a
surface renders and its controls can be reached.

**Out of scope:** logic, edge cases, error branches, validation, calculations. Those belong to the
suite, and duplicating them here creates a second thing to keep true.

This does not replace running the app for a person to look at, and it does not write tests. It
starts the app, reads it, reports, and stops it.

## Probes are bounded to dependencies with a lifecycle

Code at rest stays at rest. A frontend does not spontaneously stop existing — but a hosted database
can be deleted, paused, have its credentials rotated or hit a quota while the code is untouched, and
so can an external API's key, a cache, or an auth provider. **Those are the things that break without
a commit, and each one gets a probe.**

One probe per such dependency, not one per application. An application with a database and a mapping
API has two: checking only the database reports it healthy while routing is dead.

Where the project declares its external dependencies — an `.env.example`, a service manifest — that
declaration **is** the list, and it is maintained because new developers need it. Read it before
reading code.

## Steps

1. **Establish there is an application.** A repository with no start configuration and no
   application — a configuration store, a documents folder — gets "nothing to verify". Do not hunt
   for something to start.
   *Done: a start configuration is named, or "nothing to verify" has been reported.*

2. **Check the declared prerequisites.** Read-only: installed dependencies, declared environment
   keys, any generation step the start command assumes. A gap here is **"not ready to verify"**,
   never a defect — an absent dependency says nothing about the application.
   *Done: every declared prerequisite is present, or named as missing.*

3. **Take the control.** Run the project's test command and record the result. A failing suite does
   not stop the run; it makes attribution harder, and the report says so.
   *Done: the suite's state is recorded.*

4. **Confirm, then start.** Read the **start command** and whatever it runs on initialisation, and
   say what *starting* will touch. This is the start path only — what the application does once a
   page is requested is not knowable before starting, and belongs to the channels. **Stop if
   starting includes writes nobody sanctioned.** Where the configuration offers several targets,
   name them and ask rather than guess. Where something already holds the port, attach to it and
   say so rather than starting a second instance.
   **Let the control finish first.** A suite and an application compete for the same machine, and
   workers that time out under load look exactly like test failures.
   *Done: the application answers, the report names which instance, and the Startup writes row is
   filled.*

5. **Read every channel the environment exposes.** What the process says, what the runtime says,
   what crossed a boundary, what a person would see. **No channel is privileged** — a rendering
   fault appears in none of the others, and a dead dependency can appear in only one.
   *Done: every channel carries a value in the report; none is omitted.*

6. **Probe each lifecycle dependency**, preferring a path that writes nothing. Look first for an
   operation the application itself performs that reads before it decides.

   **Where no such path exists**, probe the dependency directly, using the application's own
   configuration and request shape — and label it **out-of-band**. The distinction is not cosmetic:
   an out-of-band probe proves the dependency is alive and proves nothing about the application's
   path to it. An application whose every route returns early without a session gives you only this.
   *Done: each dependency is recorded as answering through the application, answering out-of-band,
   or unreachable without a write.*

7. **Reproduce before reporting.** Re-run anything the channels surfaced against a minimal case the
   change never touched — a throwaway instance, a second application, an untouched route. Three
   outcomes, and the third is the one that gets skipped:
   - **The control fails identically** — it is the environment. Drop it.
   - **The control is clean and the cause traces to this application** — report it.
   - **The control is clean but nothing ties it to this application** — an error whose stack holds
     no application frames, say. Label it **unattributed** and report it as that. It is neither a
     defect nor environmental, and calling it either is a guess.

   *Done: everything surfaced is labelled defect, environmental or unattributed, and every defect
   has survived a control.*

8. **Report, then stop what was started.**
   *Done: every row carries a value, Findings included — "none" is a value and an empty row is not;
   the process is stopped; and **the port is confirmed free**, not merely the task stopped.*

Step 7 is the one that carries the weight. Without it the report is a list of things the environment
happens to do.

## Surfaces

The entry point always, plus any surface the user names or a recent change touched. Not every route:
breadth belongs to the probes, which is where a dead dependency shows up. This is a scope choice, not
a claim that the suite covers the rest — closing that gap is the suite's job.

## The report

Fill every row. A row with no value is a channel not read.

```
VERIFY — <project> — <date>

Prerequisites   ready | not ready (<what is missing>)
Control         suite passing | failing (<command>)
Startup writes  nothing beyond gitignored build output | <what starting touches>
Start           ready in Xs on :<port> | attached to existing | failed (<why>)

Channel         State
<channel>       clean (<n> noise lines excluded) | <what it showed> | n/a (<why>)

Probes
  <dependency>  through the app | out-of-band | unreachable without a write    <result>

Findings
  <anything reproduced that is not a start, dependency or surface failure> | none

Verdict         WORKING | BROKEN (<one line>) | NOT VERIFIED (<why>)
```

**Three verdicts, not a severity scale.** The outcomes are working, broken, or couldn't-tell.

**Findings exists because running a system reveals things that are not failures of the three
checks.** A route that performs database work before redirecting an unauthenticated request breaks
nothing that verify tests, and no unit test can see it. **WORKING with a non-empty Findings block is
a valid and expected result** — that combination is the point of the row, not a contradiction. Keep
each finding to a line or two; it is a pointer, not a report.

**"Clean" carries its exclusions.** A development server's console is never empty. *"clean (3
framework lines excluded)"* is honest; *"clean"* is not, if three lines were dropped. What counts as
framework noise is in [`mechanics.md`](mechanics.md), because it changes with every framework
version.

The report goes to the conversation and nowhere else. The run writes whatever the project's own test
and start commands write; that is the project's business.

## Mechanics

How to start an application and read each channel in this environment, the framework noise list, and
the observed quirks that make a control necessary: [`mechanics.md`](mechanics.md). It is separate
because it expires and this file does not.
