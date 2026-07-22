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
| `docs/PROGRESS.md` | The **warm** handoff doc. Contains the **Delivery Milestones table**; flip the milestone row to the table's own "done" token; overwrite living-state sections in place; **roll completed history to the archive** (see "Keeping docs/PROGRESS.md bounded") | **Yes** — create if absent |
| `docs/PROGRESS-ARCHIVE.md` | The **cold** append-only history (completed-milestone narratives, review logs, merge log). Not auto-loaded; consulted on demand. Roll history into it each wrap; never delete or rewrite existing entries | Created lazily on first rollup |
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
- **`gitBackend: none`:** the project has no version control — reconcile docs only, run no
  git ritual, and state "no version control — docs only." **`gitBackend: github` with no
  git repo or no working `origin`:** this is drift — STOP loudly and resolve before
  wrapping (it should not happen after `new-project`).
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

The git ritual runs only for a `gitBackend: github` project; a `none` project has no ritual
(docs only). The saved `gitWorkflow` in `.claude/wrap-it-up.json` **pre-fills** the plan but
never runs blind. Both workflows push — there is no no-push mode.

| Workflow | Actions |
|----------|---------|
| `merge-to-main` | commit remaining work on `feat/*` → `git merge --no-ff` into main → push → delete the feature branch → record the merge as prose (date + branch, **never the literal merge SHA** — see note) in the `docs/PROGRESS-ARCHIVE.md` Housekeeping log; the warm doc keeps only the pointer (see "Keeping docs/PROGRESS.md bounded") |
| `push-feature-branch` | commit → `git push -u <remote> <branch>` (no merge) |

### Remote resolution (before any push)

A push step is only ever planned or executed against a remote **proven to exist**. This
runs for every `gitBackend: github` project (a `none` project never reaches here). Before
printing any plan, resolve the configured `remote`:

- **Resolves** (`git remote get-url <remote>` succeeds — offline; confirms the remote is
  *configured*, not reachable; the post-push check below confirms the push landed) →
  include the push in the plan as normal.
- **Does not resolve → STOP.** This is drift: a GitHub-backed project has lost its remote.
  Do not push and do not downgrade to local-only (that state no longer exists). Surface
  loudly and gate, naming the configured remote:
  > Config names `gitBackend: github` with remote `<remote>`, but
  > `git remote get-url <remote>` finds no such remote. Push cannot happen.
  >
  > 1. Re-add the remote now — give me the URL (or I will `gh repo create <name> --private
  >    --source=. --remote=<remote> --push`), verify it, then push.
  > 2. Do not wrap until version control is fixed.
  >
  > Which? (1/2)

**Legacy configs (no `gitBackend`):** infer once and record it. If `origin` resolves, treat
as `github` and write `gitBackend: github`. If the config has `remote: null` or no remote,
this is a legacy local-only project — the state no longer allowed. STOP loudly every wrap
until resolved:
  > This project predates the GitHub-or-nothing policy: it has git but no GitHub remote.
  >
  > 1. Publish to GitHub now — `gh repo create <name> --private --source=. --remote=origin
  >    --push`, verify, then record `gitBackend: github`.
  > 2. Confirm it should have no remote — record `gitBackend: none`; wrap does docs only.
  >
  > Which? (1/2)

Never silently skip a push, and never report a milestone as pushed when it was not.

- If already on the main branch: **skip the merge**, just commit + push main (still gated).
- `--no-ff` is used so each milestone is a single revertable merge commit visible in
  `git log --graph`.
- **Never write the literal merge SHA into a tracked file.** A commit cannot contain its
  own hash, so the merge SHA is only known *after* the merge exists — writing it into
  docs/PROGRESS.md (a tracked file) forces a redundant follow-up commit on main
  every wrap. The SHA already lives in two authoritative places without churn:
  `git log --graph` (the record) and `.claude/wrap-it-up.json` `lastWrappedSha`
  (untracked local baseline, written in step 10). The archive's Housekeeping log records the
  merge as prose (e.g. "M5 merged --no-ff into main on 2026-07-03"), not the hash.
- Always print the plan in this format before executing — milestone wrap example
  (GitHub-backed; the remote is always confirmed before the push is listed):
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
  "gitBackend": "github",
  "gitWorkflow": "merge-to-main",
  "mainBranch": "main",
  "remote": "origin",
  "storeRoot": "<absolute path to the Lindholm Code store>",
  "lastWrappedSha": "cbb5103…",
  "lastWrappedAt": "2026-06-16"
}
```

`gitBackend` is `github` or `none`. For `github`, `remote` must name a remote that resolves
via `git remote get-url` (normally `origin`) — never a name that does not exist, since a
fabricated remote is exactly what turns "push" into a silent no-op (see Git Ritual → Remote
resolution). For `none` the git keys are omitted and no git ritual runs. There is no
local-only (git without a remote) state.

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
   "Next steps"; write the `RESUME HERE` section. **Then roll history to the archive** (see
   "Keeping docs/PROGRESS.md bounded"): relocate the previously-current milestone's narrative +
   review log — and at most one lingering legacy narrative — plus the Housekeeping merge line into
   `docs/PROGRESS-ARCHIVE.md`, leaving the warm doc describing only the just-shipped + next
   milestone. Obey the no-loss invariant (relocate, never summarize or delete).
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
11. **Session cleanup** — if `.claude/sessions/` holds exactly one `save-session` handoff file, delete it (the milestone `RESUME HERE` now supersedes it). If more than one exists, list them numbered and ask which to delete, closing with a single `Which? (1-N)` (global interaction-design doctrine); do not delete by default.
12. Print a **short** closing instruction (see below).

## Closing Instruction (keep it short)

The "safe to clear" statement is always the **last line** of the message — never
interleaved with other information. Put every status and next-step detail above it, so the
final line the user reads is the clear-to-proceed signal and nothing follows it.

The "Next session" line is a single pointer, not a menu: name the one highest-value next
step, or point to `docs/PROGRESS.md` — never enumerate two or more candidate next-works
here. Enumerating balloons the closer and tempts you to end on the guidance instead of the
clear-to-proceed signal.

Before sending, verify the final line is exactly `Safe to /clear.` with nothing after it.

After a successful wrap:

> Milestone `<name>` wrapped & merged (`<sha>`).
> Next session: read docs/PROGRESS.md → start `<next pending milestone>`.
> Safe to `/clear`.

Empty diff (nothing changed since last wrap):

> Nothing changed since last wrap.
> Safe to `/clear`.

Never invert the order — the exact failure to avoid:

> Bad:
> Milestone M13 wrapped & merged (`c4854a6`). Safe to `/clear`.
> Next session: read docs/PROGRESS.md. Highest-value next work is the ADR 0007
> validation, or M11 real-data validation of the concurrency model ...

Here "Safe to `/clear`" is buried mid-message and a multi-option next-step paragraph
follows it. The clear-to-proceed signal must be the last thing the user reads.

## Keeping docs/PROGRESS.md bounded (rollup to the archive)

docs/PROGRESS.md is the **warm** handoff doc: it describes the *present* — the current
milestone, what works now, open decisions, open future work — and stays roughly constant in
size no matter how many milestones ship. Completed history lives in the **cold**
`docs/PROGRESS-ARCHIVE.md` (append-only, tracked, never auto-loaded). Without this discipline the
warm doc grows O(n) in milestone count: the template's "overwrite in place" was never enforced,
and review logs / merge records had nowhere to go but the living doc.

**No-loss invariant.** The rollup only ever *relocates* historical content — never summarizes,
condenses, or deletes it. After a wrap, `docs/PROGRESS.md` + `docs/PROGRESS-ARCHIVE.md` together
hold every piece of *history* the warm doc held before. The exception is the **current-state
sections** — "What works right now", "How to run", "Next steps", "RESUME HERE" — which are
regenerated each wrap to describe the present, not relocated; their prior text stays recoverable
in git, since docs/PROGRESS.md is tracked. This invariant is the acceptance check for the rollup.

**On each wrap, after flipping the milestone token, roll up:**

1. **"Where we are"** keeps only the current (just-shipped) milestone in full narrative. Move the
   **previously-current** milestone's narrative — with its full review log — into the archive.
   **No backfill:** if legacy completed-milestone narratives still linger from before this
   discipline, move only the **single oldest** this wrap (the backlog drains one per wrap; do not
   mass-archive).
2. **Housekeeping merge log** → append the merge-record prose to the archive, not the warm doc.
   The warm doc keeps only the pointer line (below).
3. **"Deferred / future tasks"** → when an entry has shipped, move its line to the archive (never
   delete; git and the milestone table also record it). Open future work stays.
4. **"Confirmed decisions" / "Environment gotchas"** → stay in the warm doc while they are live
   guidance. When one is superseded, relocate the entry **verbatim** to the archive — never swap
   it for an ADR pointer or delete it (the ADR may not carry the operational "do not reintroduce
   X" phrasing).

**The archive is reachable, not a graveyard** (a rollup obligation, not optional polish):

- One dated section per milestone: `### M<n> — <name> (<YYYY-MM-DD>)`, holding that milestone's
  relocated narrative + review log + merge-log line — so a later lookup is one open + grep.
- The warm doc always carries a visible pointer (in the `## Housekeeping` slot):
  `Full history, the merge log & per-milestone review logs live in docs/PROGRESS-ARCHIVE.md`.
  The Delivery Milestones table is the always-visible index; the archive is the expand-on-demand
  detail behind any row — a reader is never *unaware* history exists, only spared its weight.
- Create `docs/PROGRESS-ARCHIVE.md` lazily, on the first wrap that has something to roll.

## docs/PROGRESS.md Template (living-state, overwrite in place; history rolls to `docs/PROGRESS-ARCHIVE.md`)

```markdown
# PROGRESS — <Project>

> Handoff/status doc. Read this (plus `CLAUDE.md` and `docs/PRD.md`) when resuming work.
> Last updated: <YYYY-MM-DD>.

## Where we are
<current milestone only — prior milestones roll to docs/PROGRESS-ARCHIVE.md>

## What works right now
<verified capabilities with evidence>

## How to run
<commands, refreshed from the verification gate>

## Confirmed decisions (do not re-litigate)
## Environment gotchas (already solved — keep these)
## Next steps (pick one)
## Deferred / future tasks (don't lose these)

## RESUME HERE — <next milestone>
<2-3 line pointer: what's done, what's next, method — milestone-level only; never a count or list of open checklist items (those live in docs/CHECKLIST.md)>

## Housekeeping
Full history, the merge log & per-milestone review logs live in `docs/PROGRESS-ARCHIVE.md`.
```

## docs/PROGRESS-ARCHIVE.md Template (append-only cold history; not auto-loaded)

```markdown
# PROGRESS ARCHIVE — <Project>

> Cold history. Not read at session start; consult on demand for past decisions and review
> findings. Append-only, newest milestone first. Never edit or delete existing entries.

### M<n> — <name> (<YYYY-MM-DD>)
<the milestone's relocated "Where we are" narrative, verbatim>
**Review log:** <code / security / design verdicts, findings + measurements, dispositions, test deltas>
Merged: <e.g. M<n> merged --no-ff into main on <YYYY-MM-DD> (branch feat/...)>
```

## Common Mistakes

- **Writing conversation claims the diff doesn't support.** Surface the conflict; don't write it.
- **Marking a milestone complete on red/unverified tests.** Record reality instead.
- **Running the git step without showing the plan.** Always gate and print first.
- **Reinventing docs/PROGRESS.md structure.** Match the existing file's sections as-is.
- **Letting "Where we are" accrete completed milestones.** Each wrap, roll the superseded
  milestone's narrative + review log into `docs/PROGRESS-ARCHIVE.md`; the warm doc describes the
  present only. Relocate, never delete (see "Keeping docs/PROGRESS.md bounded").
- **Reading or loading `docs/PROGRESS-ARCHIVE.md` at session start.** It is cold storage —
  consult it only on demand, for a specific historical decision or review finding.
- **Hardcoding the milestone status token.** Use the table's own legend (`complete`/`done`/…)
  and keep its parenthetical annotations.
- **Treating this like `/save-session`.** This is for *completed* milestones, not checkpoints.
- **Triaging the checklist.** Only commit `[x]` tasks `implement-task` finished; never scan, promote, or pick open `[ ]` items here.
- **Restating an open-items count/list in RESUME HERE or Next steps.** The open granular set lives only in `docs/CHECKLIST.md`, the single countable surface; a count copied into the note duplicates it and can only go stale (`continue-project` reads the checklist directly). State milestone-level next steps only.
