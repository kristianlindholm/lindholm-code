---
name: promote-prototype
description: Turn a finished prototype into a real product - publish it to GitHub behind a blocking pre-commit gate, then harden it milestone by milestone without changing what it does.
disable-model-invocation: true
---

# Promote Prototype

Takes a project built under prototype mode and makes it a product. Two stages, in this order:

1. **Publish** — minutes. Nothing is committed until a blocking gate confirms the tree holds
   nothing that must never reach a repository. Then the prototype lands on GitHub exactly as it is.
2. **Harden** — however long it takes. The project is an ordinary product project from that point,
   hardened one milestone at a time through the skills that already exist.

Publishing comes first because a prototype has no version control at all: until Stage 1 finishes,
the working tree is the only copy of the whole project, and Stage 2 is the phase most likely to
break something.

**What promotion may and may not change.** The implementation is replaceable — rewrite any part of
it the review says is worth rewriting. What the product *does*, and how it looks and behaves, is
not. `docs/FEATURES.md` and `docs/captures/` are the contract; they were recorded during the build
precisely so this stage has something to preserve against.

## Stage 1 — Publish

### Step 1 — Confirm the state and gather what Gate 6 skipped

Read `.claude/wrap-it-up.json`. If `mode` is not `prototype`, stop: there is nothing to promote.

Read the Delivery Milestones table in `docs/PROGRESS.md`:
- **Every milestone complete** — proceed.
- **Milestones still pending** — promoting early is allowed, but say what it means before doing it:
  the pending rows stay pending and are built afterwards with `/implement-milestone`, under full
  rigor, like any product milestone. Confirm before continuing.

A prototype never ran Gate 6, so the repository parameters were never chosen. Ask them now,
recommending the defaults and asking only about what the user wants to change:

1. Repository name — default the project folder name.
2. Visibility — default **private**.
3. Workflow — default `merge-to-main`.

Check `gh auth status`. If it is not authenticated, STOP and tell the user to run `gh auth login`,
then re-run this skill. Never fall back to a local-only repository — that state does not exist.

Done: the mode is confirmed, the milestone state is known and acknowledged, the repository
parameters are settled, and `gh` is authenticated.

### Step 2 — Initialize git and write the scaffold

Nothing is committed in this step.

1. `git init -b main`.
2. Write `.gitignore` and `.gitattributes` at the repository root, using the exact file contents
   given under those two entries in `new-project`'s "Files Created" section. That skill is the
   source of truth for both; do not improvise a variant. Their "GitHub-backed projects only" note
   describes `new-project`'s own gate, not this one — promotion is the moment the project becomes
   GitHub-backed, so both files are written here.

Writing `.gitignore` before the gate below matters: it removes `resources/`, `.claude/`, and
`.env*` from what would be staged, so the gate reports on the set that would actually be committed
rather than on everything on disk.

Done: the repository is initialized, both scaffold files are written from `new-project`'s content,
and nothing has been committed.

### Step 3 — The pre-commit gate (blocking)

This is the one step that cannot be waived, hurried, or assumed. A prototype was built without any
security review, so this is the first and only check between it and a remote. Everything after a
commit is permanent: deleting a line does not remove it from history, and a key that lands here has
to be rotated whether or not the project is ever used again.

Enumerate exactly what would be staged — `git add -A --dry-run`, or `git status --porcelain` after
the init — and examine every file in that set:

- **Secrets** — API keys, tokens, passwords, connection strings, private keys, certificates,
  service-account JSON. Grep for the high-signal patterns, then read any configuration or
  constants file rather than trusting the grep.
- **Credential files** — `.env` and anything holding the same thing under another name.
- **Real data used as fixtures.** An in-house proof of concept is often built against a live export
  of company or customer records. A fixture holding real people is as unpublishable as a key and
  far less likely to be noticed, because it looks like test data. Check what the fixtures actually
  contain.
- **Oversized or generated artifacts** — build output, dependency directories, large binaries, a
  database file. These belong in `.gitignore`, not in the first commit.

For every finding: name the file, say what is in it, and resolve it — move the value to an
environment variable, add the path to `.gitignore`, or replace real records with synthetic ones.
Then re-run the enumeration and confirm the finding is gone.

**Do not proceed to Step 4 until this passes on a fresh run, read rather than assumed.** If
anything is ambiguous, stop and ask; a false negative here is not recoverable.

Done: the exact set of files that would be committed has been examined, every finding is resolved,
and a fresh re-run is clean.

### Step 4 — Commit and publish

Follow `new-project`'s "Initialize and publish to GitHub" procedure from its step 2 onward. Its
pre-existing-file line-ending check applies in full and matters more here than anywhere else: in a
promotion *every* file is pre-existing, so this is the single moment a mixed-ending file can be
caught before git normalises it into a blob and stops being able to report it ever again.

Commit as `chore: promote prototype to product`, then create and push the remote, then **verify
with evidence** before reporting success: `git remote get-url origin` must resolve, and
`git rev-parse origin/main` must equal local `HEAD`.

Done: the prototype is on GitHub as the repository's first commit, verified by both checks — never
reported as published on the strength of a command that appeared to succeed.

### Step 5 — Record the new state

Rewrite `.claude/wrap-it-up.json`: set `mode` to `product`, `gitBackend` to `github`, and record
`gitWorkflow`, `mainBranch`, and `remote` as Gate 6 would have. Preserve `storeRoot` and any other
key this skill does not own.

Report the GitHub URL. The project is now an ordinary product project: `/implement-milestone`,
`/wrap-it-up`, and `/continue-project` all behave normally from here, and `/build-prototype` will
correctly refuse to run on it.

Stage 1 is a complete, useful stopping point. Say so, and confirm before starting Stage 2.

Done: the config records a published product project, the URL is reported, and the user has
confirmed whether to continue into hardening.

## Stage 2 — Harden

### Step 6 — Establish the behaviour baseline

Before changing anything, confirm the contract is accurate. Launch the app, walk every entry in
`docs/FEATURES.md`, and re-capture every surface in `docs/captures/`.

Anything in the feature log that does not actually work is recorded now as a known defect rather
than silently treated as a behaviour to preserve. You cannot preserve behaviour you never confirmed
was there, and a rewrite that faithfully reproduces a broken screen has preserved the wrong thing.

Done: every feature in the log is confirmed working or recorded as a known defect, and the captures
match what the app currently does.

### Step 7 — Harden one milestone at a time

Work the `docs/PROGRESS.md` milestone rows in order. Each one gets its own branch and its own pass,
and merges through `/wrap-it-up` like any other milestone.

For each milestone, run what prototype mode skipped:

- **Review** — `code-reviewer` plus the stack's reviewer, and `security-reviewer` where the work
  hits one of `code-review.md`'s security triggers. Resolve every CRITICAL and HIGH; record any
  accepted MEDIUM or LOW.
- **Tests** — backfill to the 80% coverage gate in `testing.md`. A confirmed bug finding gets a
  failing test first, through the `tdd` skill, so the fix is proven and cannot regress.
- **ADRs** — a decision made during the build that meets `new-project`'s three criteria gets an ADR
  written now, in `docs/adr/`, numbered in sequence.
- **Design review** — for a milestone with UI, check the built screens against `docs/DESIGN.md`,
  `design-quality.md`, and `design-principles.md`.
- **Re-verify the contract** — re-run that milestone's entries in `docs/FEATURES.md` and re-capture
  its surfaces. A behaviour that changed is a regression to fix, not a result to accept, unless the
  user explicitly decides otherwise.

**Scoping note.** The prototype landed as a single commit, so **there is no per-milestone diff to
review**. Scope each pass by area instead — use the milestone's scope in `docs/PROGRESS.md` and its
entries in `docs/FEATURES.md` to name the files it owns, and tell the reviewer agents to read those
files rather than a diff. State the scope explicitly when dispatching; an agent given no scope will
review the whole codebase and produce findings belonging to other milestones.

Mark each row as it merges by appending a parenthetical annotation to the status it already holds
— `complete (hardened)`. Never replace the status token itself: `wrap-it-up` owns that column, reads
the table's own vocabulary rather than a hardcoded word, and preserves parenthetical annotations.

Done: every milestone has been reviewed, tested to the coverage gate, and re-verified against the
feature log, with each pass merged and pushed.

### Step 8 — The final gate

Once every milestone is hardened, run `/security-check` — the codebase-wide audit that the Final
gate section of `docs/PROGRESS.md` has been holding open since the project was created. Tick the
box when it passes.

Done: the codebase-wide audit has passed and the Final gate checkbox is ticked. The project is a
product, with nothing outstanding from its life as a prototype.
