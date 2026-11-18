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
| `CLAUDE.md` | The **index**, plus the rules binding every change. Propose a cross-cutting rule only; anything narrower is routed to a topic file below | No — rare |
| `docs/rules/<practice>.md` | This project's rules, one file per practice. Where a genuine new convention lands | No — rare |
| `docs/architecture/<subsystem>.md` | How one subsystem is built and why. Where new architecture detail lands | No — rare |
| `docs/adr/` + its `README.md` | Where a proposed rule's reasoning lands when it needs one. Gated like the rest; the ADR and its index row go in the same diff | No — rare |

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
- **Project-rule additions → confirmation gate.** The gated surface is `CLAUDE.md`, `docs/rules/`, `docs/architecture/` and `docs/adr/` together — the gate follows the rule, not the file, so routing a rule into a topic file never routes it past the user. Show the diff and ask: "Proposed addition to `<file>`: [diff]. Add? (Y/N)" Reject = skip that edit only; continue.
- **Say where it lands, as part of the proposal.** A rule binding every change stays in `CLAUDE.md`; one binding a practice goes to `docs/rules/<practice>.md`; an architecture fact goes to `docs/architecture/<subsystem>.md`; the reasoning behind any of them goes to an ADR. Adding a topic file means adding its line to the `## Rules` index in the same diff — a file missing from the index is one nothing finds. Routing is part of what is approved, not an implementation detail decided afterwards.
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
  (untracked local baseline, written in step 13). The archive's Housekeeping log records the
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
decide what to work on next. It appends exactly one kind of new entry — a session finding the user
approved at the findings sweep (see "Findings sweep") — and that is a write, not a triage.

When `docs/CHECKLIST.md` holds `[x]` done-awaiting-commit tasks:

- Include them in the gated git ritual. Derive each commit message from the task's entry text,
  inferring the conventional type from the diff (`feat`, `fix`, `refactor`, ...).
- **Remove the `[x]` line (and its `while:` context line) from `docs/CHECKLIST.md` as part of
  the same commit** — the deletion and the work land together. This is Execution Order step 11,
  which runs before the git plan is printed, so the deletions are in the working tree by the time
  the plan is confirmed.
- **Then read the file back and confirm no `[x]` line survives.** Without that check the removal
  is the easiest thing in the wrap to skip, and skipping it inverts the commit: the one that was
  meant to close the task is the one that preserves it, in a queue every future session reads.
  Measured across two projects before this step existed — 34 `[x]` lines committed by five wraps
  and not one removed by any of the twelve commits that touched those files, while the instruction
  to remove them had been in this skill since its first commit. The read-back is the difference
  between an instruction and a step that holds.
- **Milestone also closing:** the task commit rides along with the milestone wrap; remove the
  `[x]` lines in that wrap commit.
- **No milestone advanced (task-only):** skip the milestone doc reconciliation entirely — no PRD
  read-for-context step, no docs/PROGRESS.md milestone flip, no `RESUME HERE` rewrite. **The truth
  pass still runs** (see "Truth pass over docs/PROGRESS.md, and over the rules index"). This path skips *rewriting* the warm
  doc, never *checking* it: it is the path that runs most often and the one that leaves the most
  false claims behind, precisely because it touches nothing. Otherwise just the gated commit that
  contains the task's code change plus the checklist line removal.

`docs/CHECKLIST.md` is distinct from docs/PROGRESS.md's "Deferred / future tasks": the checklist
holds granular captured side-tracks, "Deferred" holds milestone-scale future work. Do not move
items between them.

## Truth pass over docs/PROGRESS.md, and over the rules index

The warm doc is regenerated only at a milestone wrap, so every present-tense claim in it is an
assertion nothing re-checks in between. Left alone it goes false in silence: a suite count rots the
next time a test is added, a checklist claim rots the next time `/save-for-later` runs, and the
project reports neither. The rules under "Common Mistakes" say which claims not to write — they
bind the writer only at the instant of writing. This pass is what holds them afterwards, by reading
the finished document back and asking whether what it says is true.

**Runs on every wrap, task-only included.** A task-only commit skips the milestone reconciliation
(Execution Order steps 5-6); it never skips this pass.

**Step 1 — name the owning surface.** Take each present-tense claim in the warm doc: "Where we
are", "What works right now", "How to run", "Confirmed decisions", "Next steps", "RESUME HERE".
For each, name the surface that owns the fact and can settle it:

| Claim | Owning surface |
|-------|----------------|
| the suite is green | a fresh run of the test command — this wrap's verification gate |
| parked or open task state | `docs/CHECKLIST.md` |
| the codebase-wide security audit | the "Final gate" checkbox in this same file |
| a milestone's status | the Delivery Milestones table |
| the branch was pushed | the remote ref (see "After a push runs — verify it landed") |
| a measured figure (timing, size, throughput) | the wrap it was measured at, carried in the text |

A claim whose owning surface cannot be named is not verifiable. It does not get the benefit of the
doubt — see Step 2.

Done: every present-tense claim is paired with a surface, or marked unverifiable.

**Step 2 — check it and act.** Against its own surface, each claim is one of:

- **True** — leave it.
- **False** — correct it if the warm doc should carry the fact at all; delete it if it should not.
  A claim duplicating a surface that owns it is deleted, not refreshed: refreshing it only resets
  the clock on the same defect.
- **Unverifiable** — delete it. Do not soften it, and do not re-word it into something vaguer that
  survives by saying less. A claim no one can check is the one that rots unnoticed.

docs/PROGRESS.md is an autonomous surface (see "Confirmation Model"), so apply the corrections
without a gate and report in one line what changed. Never carry a claim you have found false into
the commit.

**Step 3 — check the index resolves, and report anything over the standard.** Two cheap reads,
both of which catch failures nothing else would.

*The index and the files must agree.* Every line in `CLAUDE.md`'s `## Rules` index points at a
file that exists, and every file in `docs/rules/` and `docs/architecture/` has a line. A file with
no line is invisible — it will never be opened, however good it is. A line with no file is a
promise that breaks the moment someone follows it. This is the store's "references must resolve"
rule applied to a project.

**Repairing either is a proposed edit to `CLAUDE.md`, so it goes through step 9's gate** — this
pass finds the mismatch and reports it; it does not fix it here. `docs/PROGRESS.md` is autonomous
and `CLAUDE.md` is not, and an index repair is no exception.

*Report what is over the standard.* `coding-style.md` ("Authored governance documents") bounds
these files and the entries inside them. Name any that exceed it, with its size, and stop there —
**report, never silently rewrite.** Tightening a rule is a change like any other and goes through
step 9's gate.

This check exists because a standard nothing reads back is a prose hint (`coding-style.md`
records what that cost).

Done: the index resolves in both directions or the mismatch is queued for step 9, and any file or
entry over the standard is named with its size.

Done: every present-tense claim in docs/PROGRESS.md is true, corrected, or gone.

### The suite count does not belong in the warm doc

State the green state and point at the command — "the suite is green as of this wrap; run
`<command>`" — and put the count in the archive's review log as a delta ("Suite 1032 -> 1046"),
where the PROGRESS-ARCHIVE template already carries test deltas.

The reason is not that the number rots; anchoring it would fix that. It is that a count means
nothing on its own. Its only use is as a comparison against the previous wrap, which makes it a
series, and the series already has a home — so the newest value in the warm doc duplicates a
surface that owns it. It also rots faster than anything else in the file: a count goes false every
time someone **adds** a test, which is most commits in a test-first project.

**What the count is actually for, and who does that job.** A count is the only guard against silent
test loss — a runner that stops collecting a file reports zero failures, green and worthless. That
guard is this pass, not the prose. Compare the verification gate's fresh count against the newest
delta recorded in `docs/PROGRESS-ARCHIVE.md`. Up or level is expected. **Down is a finding:**
surface it before the git step. It may be legitimate — tests removed with the code they covered —
but it is never assumed, and a milestone is not recorded as verified on a suite that lost tests
for an unexplained reason.

## Findings sweep

A finding raised during the session and left in the conversation dies with the session —
`continue-project` reads `docs/CHECKLIST.md`, not the transcript. Sweep before the git step so
anything worth keeping lands in this wrap's commit, and so the closing instruction stays a pointer
instead of a list of loose ends.

**The checklist is for tasks, not findings.** An entry is real work that sits outside the scope of
this session. That is the whole bar, and very little clears it.

A severity dot is not the bar. The milestone review dispatches `code-reviewer` and
`security-reviewer`, and those rate every finding they raise — so arriving here wearing a dot
means only that a reviewer spoke. Being flagged and being a task are different things, and
treating the first as evidence of the second is what fills a checklist with review output.

**Routing — take the first branch that fits.**

- **Something you could just do now** — do it, now, in this wrap. The sweep runs before the git
  step, so the fix lands in this wrap's commit alongside everything else. Expect most findings to
  take this branch: a thing small enough to park is usually small enough to finish, and finishing
  it costs one edit against a queue entry that every future session reads and steps over.
- **A governing-document gap** — a standing rule the project should carry and does not — belongs
  to step 9's proposed-additions path, which carries its own show-diff-and-confirm gate. Do not
  bring it here. Name where it lands as part of the proposal: `CLAUDE.md` only if it binds every
  change, otherwise `docs/rules/<practice>.md` or `docs/architecture/<subsystem>.md`, with its
  line in the `## Rules` index.
- **Real work, outside this session's scope** — too large to absorb into the wrap, or it belongs
  to a milestone that is not this one. Only this branch reaches the gate below.
- **Everything else** — drop it. If it is not worth doing now, it is almost never worth doing
  later: the judgement that defers it today is the same judgement that will skip it every session
  from here, and the entry outlives the context that made it legible.

**Not tasks — drop these; never propose them.** Each shape below has reached a real checklist:

| Shape | How it reads | What to do |
|---|---|---|
| A measurement | "one `persist()` of a hundred 600-step boards costs about 180 ms" | it is a result — the review log in `docs/PROGRESS-ARCHIVE.md`, if anywhere |
| A disclosed ceiling | "these three builds carry no bill — bounded by the schema, not a problem" | nothing; it was measured and it was fine |
| A limit of what was verified | "the clipboard tier is verified against Excel only" | the review log, if anywhere |
| A probe | "MEASURE the autosave cost with many boards", "look at the test document in resources" | run it now, or drop it |
| A coverage note | "no fixture varies how many lanes one step names beyond six" | write the fixture now, or drop it |

A probe is the clearest case: an entry asking a future session to go and look at something is
work that takes about as long as writing the entry did.

**The one-line test.** The format below is one line plus a `while:` clause. If an entry needs a
paragraph to argue that it belongs, that paragraph is the finding and there is no task underneath
it — drop it, or do the work now. Entries have reached twenty-five lines on this route, and each
one was a review finding wearing a checkbox.

**The gate — one finding per turn.** For each proposed entry, in its own message:

1. State in plain language what was found and why it outlives this session. The entry itself is
   one line and loses the reasoning; the decision needs the reasoning.
2. Show the exact entry that would be written.
3. Close with `Add to the checklist? (Y/N)` (global interaction-design doctrine).

Never batch several findings behind one prompt, and never write an entry before its own Y.

**On N, drop it.** Record it nowhere. A live problem resurfaces from the product on its own; a
note about a past state does not earn a place in the queue every future session reads.

**Entry format** — identical to `save-for-later`'s, which owns this format; `give-feedback` writes
it too. If the format changes, change it there first and propagate.

```markdown
- [ ] 2026-08-14 | the sed -i route in CLAUDE.md strips CR from CRLF files
      while: wrapping the fully-manual planning task
```

Use today's date; the `while:` clause names the wrap this surfaced in. Append to the end of the
file, never overwriting existing entries.

**Appending is not triaging.** A new `[ ]` entry the user has just approved is not a scan, a
promotion, or a pick from the backlog (see "What this skill does not do"). That prohibition
governs the open items already on the list, which this step never reads for prioritisation.

**Most wraps propose nothing.** Silence is the ordinary outcome, not a sign the sweep was skipped:
the findings were fixed in the wrap or dropped, which is where the routing sends nearly all of
them. When nothing clears the bar, this step says nothing — no prompt, and no mention in the
closer. A wrap that proposes an entry every time is misreading the bar, not finding more work.

## Persistence — `.claude/wrap-it-up.json`

Asked **once** on first run (detected where possible), then always visible/overridable per run.
`lastWrappedSha` / `lastWrappedAt` are updated after each successful wrap.

```json
{
  "gitBackend": "github",
  "mode": "product",
  "gitWorkflow": "merge-to-main",
  "mainBranch": "main",
  "remote": "origin",
  "storeRoot": "<absolute path to the Lindholm Code store>",
  "lastWrappedSha": "cbb5103…",
  "lastWrappedAt": "2026-06-16"
}
```

`mode` is `product` or `prototype` — written by `new-project` Gate 6 and flipped by
`promote-prototype` when a prototype is published. This skill preserves it and does not act on it.

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
7. **Truth pass** — read docs/PROGRESS.md back and check every present-tense claim against the
   surface that owns it; correct or delete what fails. Then check the `## Rules` index resolves
   both ways, and report any file or entry over the `coding-style.md` standard (see "Truth pass
   over docs/PROGRESS.md, and over the rules index").
   This step runs for every wrap, task-only commits included — steps 5-6 are skipped there, this
   one is not.
8. **Graphify update (conditional):** Check whether graphify graph files exist in the project
   (look for `.graphify_chunk_*.files.txt` or a `.graphify_version` file at the project root).
   If found, invoke the graphify update workflow before the git step so that the updated graph
   is included in the milestone commit.
9. If there are proposed rule additions → name the file each lands in → show diff → confirm.
10. **Findings sweep** — route this session's outstanding findings: fix what this wrap can absorb,
    drop what is not real work, and propose a checklist entry only for work that sits outside this
    session's scope. Most wraps propose nothing. Each proposal is gated on its own (see "Findings
    sweep").
11. **Checklist finalization** — open `docs/CHECKLIST.md` (if present) and delete every `[x]` line
    together with its indented `while:` line, so the deletions are already in the working tree when
    the git plan is printed. Then **read the file back and confirm no `[x]` survives** (see
    "Checklist task commits"). Read the file here rather than trusting the note taken at step 2 —
    nine steps is too far to carry it. This step runs for every wrap, task-only commits included.
    With no `[x]` lines present it is silent.
12. **Resolve the remote** (see Git Ritual → Remote resolution) → print the git plan → confirm →
    execute (or skip) → if a push ran, **verify it landed** before reporting success (see "After a
    push runs — verify it landed"). Step 11's deletions ride in this commit alongside the work.
13. Update `.claude/wrap-it-up.json` (new `lastWrappedSha`, `lastWrappedAt`).
14. **Session cleanup** — if `.claude/sessions/` holds exactly one `save-session` handoff file, delete it (the milestone `RESUME HERE` now supersedes it). If more than one exists, list them numbered and ask which to delete, closing with a single `Which? (1-N)` (global interaction-design doctrine); do not delete by default.
15. Print a **short** closing instruction (see below).

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

> Handoff/status doc. Read this (plus `CLAUDE.md`, `docs/PRD.md`, and the topic files
> `CLAUDE.md`'s Rules index names for the work in hand) when resuming work.
> Last updated: <YYYY-MM-DD>.

## Delivery Milestones
<the milestone table from new-project — preserved across wraps; only status tokens change>

## Final gate
<the codebase-wide security-audit checkbox — preserved across wraps, never regenerated or
removed. wrap-it-up does not tick it and does not trigger it: `continue-project` surfaces it
once every milestone is complete, and `/security-check` ticks it>

## Where we are
<current milestone only — prior milestones roll to docs/PROGRESS-ARCHIVE.md>

## What works right now
<verified capabilities — what works and how it was verified; any figure carries the wrap it was
measured at, never a bare present-tense count>

## How to run
<commands, refreshed from the verification gate>

## Confirmed decisions (do not re-litigate)
## Environment gotchas (already solved — keep these)
## Next steps (pick one)
## Deferred / future tasks (don't lose these)

## RESUME HERE — <next milestone>
<2-3 line pointer: what's done, what's next, method — milestone-level only. Every present-tense claim written here must survive the truth pass: no suite count (state the green state, point at the command; the count is a delta in the archive's review log), no other figure stated as currently true, and no claim about what `docs/CHECKLIST.md` contains — empty, counted or listed alike>

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
- **Triaging the checklist.** Only commit `[x]` tasks `implement-task` finished; never scan, promote, or pick open `[ ]` items here. Appending a new `[ ]` entry the user approved at the findings sweep is not triaging (see "Findings sweep").
- **Writing a figure as currently true where nothing updates it.** The warm doc's current-state
  sections are regenerated only at a milestone wrap, so "999 tests pass" turns false the moment a
  task changes the count and nothing corrects it. A figure that might be stale is worse than none —
  its only use is as a baseline, and an untrustworthy baseline sends the reader to run the command
  anyway. Anchor it or omit it: "at the M35 wrap, 1022 tests passed" stays true, and belongs in the
  archive's review log, which already carries test deltas. In the warm doc, point at the command
  that produces the number. A standing figure already sitting there is corrected or deleted by the
  truth pass — this rule stops new ones, that pass clears the old.
- **Restating what `docs/CHECKLIST.md` contains, in RESUME HERE or Next steps.** The instance of the
  above that bites most often: the open granular set lives only in `docs/CHECKLIST.md`, the single
  countable surface, and `continue-project` reads it directly. Counted, listed and **empty** are one
  mistake, not three — "the checklist has no open items" is the form written most often, because it
  is true when written and reads as reassurance, and it turns false the first time
  `/save-for-later` runs. State milestone-level next steps only.
- **Leaving a claim in the warm doc that this wrap never checked.** Every rule above binds the
  writer at the instant of writing and nothing afterwards; the truth pass is what holds them once
  the document exists. Skipping it — or running it only on the milestone path, where the doc is
  being rewritten anyway — is how docs/PROGRESS.md rots while every rule against rot is in place.
