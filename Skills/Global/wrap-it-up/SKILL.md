---
name: wrap-it-up
description: Close out a completed milestone — reconcile the living docs (PRD, docs/PROGRESS.md, README, CLAUDE.md) with what shipped and perform the milestone git ritual. Not for mid-task checkpoints — use /save-session.
disable-model-invocation: true
---

# Wrap It Up

## Overview

Milestone-**completion** skill. When a phase of work is done, this reconciles the
repository's living documentation with what actually shipped, performs the milestone
git ritual, and leaves a clean handoff so you can `/clear` and start a *new* phase.

**Core principle:** the git diff is ground truth; the conversation is narrative color.
Never write a claim into the source of truth that the diff cannot confirm.

## When to Use

- A milestone / major feature is **complete** and you want to start something new
  (e.g. backend done → starting frontend) and clear the session in between.

**When NOT to use** — work is mid-task and you want to resume the *same* thread later.
That is a checkpoint, not a wrap. Use **`/save-session`** instead. This skill actively
redirects you there if it detects a non-clean boundary (see Boundary Guard).

| Situation | Skill |
|-----------|-------|
| Milestone complete, clearing session, new phase next | **wrap-it-up** |
| Work incomplete, resuming same work next session | `/save-session` |

## Files In Scope

| File | Handling | Mandatory? |
|------|----------|------------|
| PRD (`docs/PRD.md`) | Reconcile requirements sections (target users, success criteria, constraints); read for context — do not write milestone status here | **Yes** — create if absent |
| `docs/PROGRESS.md` | Contains the **Delivery Milestones table**; flip the milestone row to the table's own "done" token; overwrite living-state sections in place (see template) | **Yes** — create if absent |
| `docs/DESIGN.md` (+ `docs/styleboard.html`) | UI projects only. If a UI milestone changed the visual language, reconcile the tokens/sections with what shipped and regenerate the styleboard; the diff is ground truth | No — discovered |
| `README.md`, `CHANGELOG.md`, other docs | Update only if affected; never invent | No — discovered |
| `CLAUDE.md` | Propose genuine new conventions only; **the one gated file** | No — rare |

The PRD is a single file (`docs/PRD.md`). It is a requirements document —
read it for context but do not add or modify milestone status there.

**The Delivery Milestones table lives in docs/PROGRESS.md.** Use the table's own status
vocabulary — never hardcode. Read the legend (e.g. `Status: pending | in-progress | complete`)
and use its terminal token (`complete`, `done`, `shipped`, …) when closing a milestone.
**Preserve any parenthetical annotation** in the cell (e.g. `complete (engine backend)`) —
update it, don't discard it. If the table has no legend, default to `complete` and match
the casing of existing cells.

## Source of Truth & Baseline

- Ground truth = `git diff` since `lastWrappedSha` (read from `.claude/wrap-it-up.json`).
- **First run** (no `lastWrappedSha`): diff since the last `--no-ff` merge commit on the
  main branch. If none exists, fall back to conversation + working-tree state **with a
  note that doc accuracy is unverified**.
- **No git repo:** offer to `git init`. If declined, proceed conversation-only with a
  **loud accuracy warning**.
- Any claim the diff cannot confirm is **surfaced to the user, never silently written**.

## Confirmation Model

- docs/PROGRESS.md, README, CHANGELOG → applied **autonomously** (all reversible via git).
- **PRD → read for context only.** No writes.
- **CLAUDE.md additions → confirmation gate.** Show the diff and ask: "Proposed CLAUDE.md additions: [diff]. Add? (Y/N)" Reject = skip that edit only; continue.
- **Git operation → its own explicit gate.** Always print the commit message(s) and git actions in full before executing — even when a default workflow is saved. See the Git Ritual section for the exact format. Awareness over automation.

## Verification Gate (soft)

Before reconciling, establish that the milestone actually works:

1. Read the milestone's TDD checklist / planning artifacts.
2. Run the detected test/build command; capture results (test counts, build status).
3. **Flag anything not confirmable.** Green → proceed. Red → wrapping is still allowed,
   but failures are written honestly into docs/PROGRESS.md "Next steps" / "Environment gotchas".

Never refuse on red tests alone — but never record a milestone as complete on unverified work.

## Boundary Guard

If this is **not** a clean milestone boundary — incomplete TDD checklist, red tests with no
acknowledgment, or **multiple milestones marked `in-progress`** — stop and ask:

> This looks mid-milestone, not a completion. You probably want `/save-session`. Wrap anyway? (Y/N)

On override: proceed, but mark **all** milestones honestly (no false "complete"); a partial
milestone stays `in-progress` with its remaining work in "Next steps".

**Exception — task-only commit.** If no milestone is being completed in this wrap but
`docs/CHECKLIST.md` holds `[x]` done-awaiting-commit tasks, this is a finished task ready to
commit, not a mid-milestone checkpoint — even if a milestone is otherwise mid-flight. Skip the
redirect and take the task-commit path (see "Checklist task commits"); do not push the user to
`/save-session`.

## Git Ritual

The saved `gitWorkflow` in `.claude/wrap-it-up.json` **pre-fills** the plan but never runs blind.

| Workflow | Actions |
|----------|---------|
| `merge-to-main` | commit remaining work on `feat/*` → `git merge --no-ff` into main → push → delete the feature branch → record the merge in docs/PROGRESS.md Housekeeping as prose (date + branch, **never the literal merge SHA** — see note) |
| `push-feature-branch` | commit → `git push -u <remote> <branch>` (no merge) |
| `commit-only` | commit; no push/merge |

### Remote resolution (before any push)

A push step is only ever planned or executed against a remote **proven to exist**. Before
printing any plan that includes a push, resolve the configured `remote`:

- **`remote` is `null` or absent** → the project is local-only. Never list "push" in the
  plan. `merge-to-main` degrades to commit + `--no-ff` merge then delete the feature branch
  locally, **no push**; `commit-only` is unchanged; `push-feature-branch` is invalid here —
  fall back to `commit-only` and note it. State plainly: "local-only — no push."
- **`remote` is a non-null name** → verify it resolves with `git remote get-url <remote>`
  (offline; confirms the remote is *configured*, not that it is reachable — the post-push
  check below confirms the push actually landed):
  - **Resolves** → include the push in the plan as normal.
  - **Does not resolve → STOP.** Do not push — a push to a missing remote is a no-op or
    error, never a success. Surface loudly and gate, naming the actual configured remote:
    > Config names remote `<remote>`, but `git remote get-url <remote>` finds no such remote.
    > Push cannot happen.
    >
    > 1. Add a remote now — give me the URL; I will `git remote add <remote> <url>`, verify it, then push.
    > 2. Proceed local-only this wrap — commit/merge without pushing and set `remote: null` in `.claude/wrap-it-up.json` so this stops recurring.
    >
    > Which? (1/2)

Never silently skip the push, and never report a milestone as pushed when it was not.

- If already on the main branch: **skip the merge**, just commit + push main (still gated).
- `--no-ff` is used so each milestone is a single revertable merge commit visible in
  `git log --graph`.
- **Never write the literal merge SHA into a tracked file.** A commit cannot contain its
  own hash, so the merge SHA is only known *after* the merge exists — writing it into
  docs/PROGRESS.md (a tracked file) forces a redundant follow-up commit on main
  every wrap. The SHA already lives in two authoritative places without churn:
  `git log --graph` (the record) and `.claude/wrap-it-up.json` `lastWrappedSha`
  (untracked local baseline, written in step 10). Housekeeping records the merge as prose
  (e.g. "M5 merged --no-ff into main on 2026-07-03"), not the hash.
- Always print the plan in this format before executing — milestone wrap example
  (remote-confirmed form; on a local-only project omit the push line and the merge line
  reads "Merge feature branch into main (local-only — no push)"):
  > Wrapping milestone "Authentication":
  >
  > Commit message:
  > - `feat: add JWT authentication and session management`
  >
  > Git actions:
  > - Merge feature branch into main
  > - Push to remote (`<remote>` — verified)
  > - Delete feature branch
  >
  > Proceed? (Y/N)

  Task-only commit example (one or more `[x]` tasks, no milestone closing):
  > Committing 2 completed tasks:
  >
  > Commit messages:
  > - `fix: results screen empty state reads as an error`
  > - `fix: button hover state missing on mobile`
  >
  > Git actions:
  > - Commit to current branch
  >
  > Proceed? (Y/N)

### After a push runs — verify it landed

Confirm success from **evidence** before reporting "pushed": a non-error exit **and** the
remote-tracking ref now matches what was pushed — `git rev-parse <remote>/<branch>` equals
local `HEAD` (git updates that ref on a successful push, so this is reliable immediately
after). Only then report "pushed." If the push failed, report the actual state and write
**no** doc claiming the milestone was pushed. This is "Verification Before Completion"
(`~/.claude/rules/development-workflow.md`) applied to the outward-facing push.

## Checklist task commits

`docs/CHECKLIST.md` (granular tasks captured by `save-for-later` and `give-feedback`, executed by
`implement-task`) is **finalized** here, never **triaged** here. This skill only commits tasks
`implement-task` already completed; it does not scan the backlog, promote open `[ ]` items, or
decide what to work on next.

When `docs/CHECKLIST.md` holds `[x]` done-awaiting-commit tasks:

- Include them in the gated git ritual. Derive each commit message from the task's entry text,
  inferring the conventional type from the diff (`feat`, `fix`, `refactor`, ...).
- **Remove the `[x]` line (and its `while:` context line) from `docs/CHECKLIST.md` as part of
  the same commit** — the deletion and the work land together.
- **Milestone also closing:** the task commit rides along with the milestone wrap; remove the
  `[x]` lines in that wrap commit.
- **No milestone advanced (task-only):** skip the milestone doc reconciliation entirely — no PRD
  read-for-context step, no docs/PROGRESS.md milestone flip, no `RESUME HERE` rewrite. Just the gated
  commit that contains the task's code change plus the checklist line removal.

`docs/CHECKLIST.md` is distinct from docs/PROGRESS.md's "Deferred / future tasks": the checklist
holds granular captured side-tracks, "Deferred" holds milestone-scale future work. Do not move
items between them.

## Persistence — `.claude/wrap-it-up.json`

Asked **once** on first run (detected where possible), then always visible/overridable per run.
`lastWrappedSha` / `lastWrappedAt` are updated after each successful wrap.

```json
{
  "gitWorkflow": "merge-to-main",
  "mainBranch": "main",
  "remote": "<remote name that resolves, or null for local-only>",
  "storeRoot": "<absolute path to the Lindholm Code store>",
  "lastWrappedSha": "cbb5103…",
  "lastWrappedAt": "2026-06-16"
}
```

`remote` must name a remote that resolves via `git remote get-url`, or be `null` for a
local-only project. Never a name that does not exist — a fabricated remote is exactly what
turns "push" into a silent no-op (see Git Ritual → Remote resolution).

Preserve any keys already in the file that this skill does not own — notably `storeRoot`
(written by `new-project` / `continue-project`). Update `lastWrappedSha` / `lastWrappedAt`;
leave the rest untouched.

## Execution Order

1. Read `.claude/wrap-it-up.json` (run first-run setup if absent). Compute baseline → diff.
2. Read PRD, docs/PROGRESS.md, plans, and TDD checklists. Also read `docs/CHECKLIST.md` if present
   and note any `[x]` done-awaiting-commit tasks (see "Checklist task commits").
3. **Verification gate** — run tests/build; flag unconfirmables. This gate runs for every commit,
   task-only commits included (the working tree may have moved since `implement-task` ran).
4. **Boundary guard** — redirect to `/save-session` if not a clean boundary (override → honest
   labels). Exception: for a task-only commit (no milestone completing this wrap, `[x]` tasks
   present), do not prompt for `/save-session` — proceed directly to step 5.
5. If a milestone is in progress: identify it and draft all doc reconciliations, surfacing any
   diff/claim conflicts. If this is a task-only commit, skip steps 5-6 — there is no milestone to
   reconcile.
6. Apply PRD + docs/PROGRESS.md + README/CHANGELOG edits autonomously. Flip the milestone to the
   table's own terminal token (preserving its annotation); the next `pending` row becomes
   "Next steps"; write the `RESUME HERE` section.
7. **Graphify update (conditional):** Check whether graphify graph files exist in the project
   (look for `.graphify_chunk_*.files.txt` or a `.graphify_version` file at the project root).
   If found, invoke the graphify update workflow before the git step so that the updated graph
   is included in the milestone commit.
8. If CLAUDE.md has proposed additions → show diff → confirm.
9. **Resolve the remote** (see Git Ritual → Remote resolution) → print the git plan → confirm →
   execute (or skip) → if a push ran, **verify it landed** before reporting success (see "After a
   push runs — verify it landed"). If `[x]` checklist tasks are being committed, the plan includes
   removing their lines from `docs/CHECKLIST.md` in the same commit (see "Checklist task commits").
10. Update `.claude/wrap-it-up.json` (new `lastWrappedSha`, `lastWrappedAt`).
11. **Session cleanup** — if `.claude/sessions/` holds exactly one `save-session` handoff file, delete it (the milestone `RESUME HERE` now supersedes it). If more than one exists, list them and ask which to delete; do not delete by default.
12. Print a **short** closing instruction (see below).

## Closing Instruction (keep it short)

The "safe to clear" statement is always the **last line** of the message — never
interleaved with other information. Put every status and next-step detail above it, so the
final line the user reads is the clear-to-proceed signal and nothing follows it.

After a successful wrap:

> Milestone `<name>` wrapped & merged (`<sha>`).
> Next session: read docs/PROGRESS.md → start `<next pending milestone>`.
> Safe to `/clear`.

Empty diff (nothing changed since last wrap):

> Nothing changed since last wrap.
> Safe to `/clear`.

## docs/PROGRESS.md Template (living-state, overwrite in place)

```markdown
# PROGRESS — <Project>

> Handoff/status doc. Read this (plus `CLAUDE.md` and `docs/PRD.md`) when resuming work.
> Last updated: <YYYY-MM-DD>.

## Where we are
<milestone status prose>

## What works right now
<verified capabilities with evidence>

## How to run
<commands, refreshed from the verification gate>

## Confirmed decisions (do not re-litigate)
## Environment gotchas (already solved — keep these)
## Next steps (pick one)
## Deferred / future tasks (don't lose these)

## RESUME HERE — <next milestone>
<2-3 line pointer: what's done, what's next, method>

## Housekeeping
```

## Common Mistakes

- **Writing conversation claims the diff doesn't support.** Surface the conflict; don't write it.
- **Marking a milestone complete on red/unverified tests.** Record reality instead.
- **Running the git step without showing the plan.** Always gate and print first.
- **Reinventing docs/PROGRESS.md structure.** Match the existing file's sections as-is.
- **Hardcoding the milestone status token.** Use the table's own legend (`complete`/`done`/…)
  and keep its parenthetical annotations.
- **Treating this like `/save-session`.** This is for *completed* milestones, not checkpoints.
- **Triaging the checklist.** Only commit `[x]` tasks `implement-task` finished; never scan, promote, or pick open `[ ]` items here.
