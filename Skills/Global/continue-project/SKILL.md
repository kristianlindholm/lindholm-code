---
name: continue-project
description: Integrity check before any session work begins — establish where the project stands before starting.
disable-model-invocation: true
---

# Continue Project

Establishes where a project stands — session state, language coverage, and git integrity — before any new work begins.

Do not skip this. Do not start coding before it completes.

## Step 1 — Read CLAUDE.md

Orient to project conventions, tech stack, and project-specific rules. These apply to everything done in this session. Note the Project Layout section: product code lives under `product/`, governing documents under `docs/`, user-supplied input under `resources/`. Confirm those three folders exist; if the project predates this layout (docs at the root), surface it rather than assuming the new paths.

Done: project conventions, stack, and rules are loaded and will govern this session, and the expected folder layout is confirmed or its absence noted.

## Step 2 — Check for new languages

Run this check once, at session start. Do not re-check mid-session.

List what is currently activated in `.claude/rules/`, `.claude/skills/`, and `.claude/agents/`. Then scan `product/` (where the product code and its manifests live) for these language indicator files:

| Indicator | Language |
|---|---|
| `package.json`, `tsconfig.json`, `*.ts`, `*.tsx` | TypeScript |
| `*.tsx`, `*.jsx` with React imports | React |
| `requirements.txt`, `pyproject.toml`, `*.py` | Python |
| `go.mod` | Go |
| `Cargo.toml` | Rust |
| `*.swift`, `Package.swift` | Swift |
| `build.gradle`, `*.kt` | Kotlin |
| `pom.xml`, `*.java` | Java |
| `composer.json`, `*.php` | PHP |
| `Gemfile`, `*.rb` | Ruby |
| `pubspec.yaml`, `*.dart` | Dart / Flutter |

For each detected language, check whether its ruleset, skills, and agents are already present under `.claude/`. If everything is present, there is no gap.

If a gap is found, ask once:
> "The project appears to use [language] but [ruleset / skill / agent] is not activated. Activate now? (Y/N)"

On no: proceed — do not ask again this session.

On yes: activate the missing artifacts by performing `/new-project`'s activation procedure (Gate 2, steps 1-4) for each detected-but-missing language. Map each detected language to its row in that skill's tech-choice table, and source the copies from the store root (see below). That procedure is the single source of truth for the copy: which artifacts map to which language, where they land (`.claude/rules/<lang>/<topic>.md` with the `Stack_specific/` marker dropped), deduplication, and skip-on-no-match.

Resolve the store root once: read `storeRoot` from `.claude/wrap-it-up.json`. If it is absent, ask the user for the absolute path to the Lindholm Code store (where this configuration repository lives), use it, and write it back into `.claude/wrap-it-up.json` so it is asked only once.

Done: every detected language is either already activated or has been offered activation once, and any accepted activation has been copied in.

## Step 3 — Determine project state

First, check `.claude/sessions/` for `save-session` handoff files:
- Exactly one: read it and prepare to suggest resuming that work.
- More than one: read the most recent, list the others, and ask which to resume.
- None: continue with milestone state below.

Then check `docs/CHECKLIST.md` for parked tasks (created by `save-for-later` and `give-feedback`):
- Present: read it. Note the open `[ ]` items and any `[x]` items that are done and awaiting commit.
- Absent or empty: no parked tasks to surface.

If docs/PROGRESS.md has no RESUME HERE section:
- This is the first session after `/new-project`.
- Read the Delivery Milestones table in docs/PROGRESS.md — identify Milestone 1.
- Report: "First session. Milestone 1 is [name]." Recommend `/implement-milestone` to begin it, and stop here.

If docs/PROGRESS.md has a RESUME HERE section:
- Read that section.
- Read `.claude/wrap-it-up.json` — note `lastWrappedSha` and `lastWrappedAt`.

Done: the project is classified as first-session, resumable-from-session-file, or resumable-from-milestone, with the relevant file read; and any parked checklist tasks are noted.

## Step 4 — Check git integrity

Run this only when the project uses git and `lastWrappedSha` is set:
- No git repository: note it, skip drift detection, and proceed — there is no baseline to compare against (the same stance `wrap-it-up` takes).
- `lastWrappedSha` not set (no wrap has happened yet): skip the diff; `git status` alone is enough.

When the baseline exists:

Run: `git status`
Run: `git diff <lastWrappedSha>`

Evaluate:
- Are there uncommitted changes?
- Has the working tree drifted from the last wrapped state?
- Do uncommitted changes match the current milestone scope, or are they unexpected?

Done: the working tree is classified as clean or drifted, with drifting files named — or the git checks are explicitly skipped with the reason stated.

## Step 5 — Report and wait

Clean state: print the current milestone and what is next. Recommend `/implement-milestone` to execute the next milestone to best practice. Wait for direction.

If `docs/CHECKLIST.md` holds open `[ ]` tasks: list them, numbered by open-item ordinal (count only `[ ]` items, skipping any `[x]` entries — this is how `/implement-task` resolves `<n>`), as a pickable option alongside the milestone path, and offer `/implement-task <n>` to execute one. The milestone path stays primary; the checklist is an alternative, not a replacement. These are granular parked side-tracks, distinct from docs/PROGRESS.md's milestone-scale "Deferred / future tasks". If any `[x]` done-awaiting-commit tasks exist, note them once: "done, awaiting commit — run `/wrap-it-up`".

If a saved session was found in Step 3: lead the report with it — summarize its Exact Next Step and recommend resuming that work before anything else.

Drift detected: describe the mismatch specifically — what changed, where, and how it differs from the documented state. Propose how to reconcile. Do not begin new work until the state is resolved or the drift is explicitly acknowledged by the user.

Done: the current state and recommended next action are reported, and no new work has begun.

## What counts as drift

- Uncommitted changes not related to the current milestone
- Files modified after `lastWrappedSha` that are not part of ongoing work
- Documented milestone status that does not match the actual codebase state

Uncommitted work-in-progress on the current milestone is not drift — surface it but do not block on it.
