---
name: new-project
description: Sequential, user-confirmed gates before any code is written — requirements, architecture, design, milestones, git.
disable-model-invocation: true
---

# New Project

Sequential gates before any code is written. Each gate produces a concrete output and requires explicit user confirmation before the next gate opens. Gate 3 (Design Foundation) runs only for projects with a user-facing UI; all others are universal.

Do not write code. Do not skip or combine gates.

Every gate asks the user one decision at a time and waits — follow the global
interaction-design doctrine: present options as a numbered list closing with a single
`Which? (1-N)`, and never batch several questions into one prompt.

## Directory Layout

Every project this skill scaffolds uses one canonical layout. It separates authored
governance, generated code, and user-supplied input so the repository root stays clean:

```
project-root/
  CLAUDE.md          root - the harness auto-loads it; a thin pointer into docs/
  .gitignore         root
  .claude/           config: rules/, skills/, agents/, wrap-it-up.json, sessions/, scratch/
  product/           the actual product code (package.json / pyproject / Cargo.toml / etc. live here)
  docs/              authored governing documents
    PRD.md
    PROGRESS.md
    CHECKLIST.md     granular parked tasks and tactical refinements (save-for-later / give-feedback)
    DESIGN.md        UI projects only
    styleboard.html  UI projects only - generated render, flat beside DESIGN.md
    adr/
      0001-slug.md
  resources/         user-supplied input material (starting docs, design references, inspiration, spreadsheets)
```

Harness constraint — do not move these: Claude Code discovers `CLAUDE.md` and project
`.claude/` settings by walking up from the working directory, so `CLAUDE.md`,
`.gitignore`, and `.claude/` must stay at the repository root. `CLAUDE.md` stays at root
and points into `docs/`; it is not moved into `docs/`. Everything the gates below produce
lands under `docs/` (governance) or `product/` (code), never loose in the root.

## Gate 1 — Requirements

Before drafting anything, invoke the `grill-me` skill to interview the user. Lead with
understanding the end-user's need, then judge whether this is the right thing to build — do
not infer the product from a one-line prompt. Grill until these are concrete:

1. Who is it for — the specific users, not a category.
2. What pain points or needs it solves — the real problem the product exists to serve.
3. Is this the right thing to build — does that need justify this product, or would a
   different (or no) product serve the user better? Challenge the premise; do not accept a
   hypothetical problem.
4. What success looks like — how you will know the need has been met.
5. Known constraints — technical, legal, time, budget, or otherwise.

Then produce docs/PRD.md capturing the outcome of that interview:
- Problem / end-user need: the pain points the product exists to serve
- Target users: who specifically will use this
- Why build this: what makes it the right thing to build for that need
- Success criteria: what "done" looks like for the product
- Known constraints: technical, legal, time, budget, or otherwise

Present the draft. Wait for confirmation.

## Gate 2 — Architecture

Define:
- Tech stack with rationale for each choice
- Architectural shape: how the system is structured at a high level
- Key integration points: APIs, databases, authentication, external services

For each significant decision, evaluate all three ADR criteria:
1. Hard to reverse — the cost of changing course later is meaningful
2. Surprising without context — a reader would wonder "why did they do it this way?"
3. A real trade-off — genuine alternatives existed and one was chosen for specific reasons

If all three apply, create docs/adr/0001-slug.md (create the docs/adr/ directory if it does not exist). ADR format: title, then 1-3 sentences covering context, decision, and why. Sequential numbering.

Present the architecture summary. Wait for confirmation.

After Gate 2 is confirmed, activate rulesets, skills, and agents for the project.

First, resolve the store root: ask the user for the absolute path to the Lindholm Code
store (where this configuration repository lives). Use it for every copy below — written
as `<storeRoot>` — and record it in `.claude/wrap-it-up.json` at Gate 5 so later sessions
do not have to ask again.

1. Map each confirmed tech choice using this table:

   | Tech choice | Ruleset directory | Skills | Agents |
   |---|---|---|---|
   | TypeScript / JavaScript | `Stack_specific/typescript/` | `frontend-patterns`, `backend-patterns` | `typescript-reviewer`, `typescript-build-resolver` |
   | React | `Stack_specific/react/` + `Stack_specific/typescript/` + `Stack_specific/design/` | `react-patterns`, `frontend-patterns`, `react-testing` | `react-reviewer`, `react-build-resolver` |
   | Angular | `Stack_specific/angular/` + `Stack_specific/typescript/` + `Stack_specific/design/` | `angular-developer`, `frontend-patterns` | `typescript-reviewer`, `typescript-build-resolver` |
   | Vue | `Stack_specific/vue/` + `Stack_specific/typescript/` + `Stack_specific/design/` | `frontend-patterns`, `vite-patterns` | `typescript-reviewer`, `typescript-build-resolver` |
   | Nuxt | `Stack_specific/nuxt/` + `Stack_specific/typescript/` + `Stack_specific/design/` | `nuxt4-patterns`, `frontend-patterns`, `vite-patterns` | `typescript-reviewer`, `typescript-build-resolver` |
   | Python | `Stack_specific/python/` | `python-patterns`, `python-testing` | `python-reviewer` |
   | FastAPI | `Stack_specific/python/` | `python-patterns`, `fastapi-patterns`, `python-testing` | `python-reviewer`, `fastapi-reviewer` |
   | Django | `Stack_specific/python/` | `python-patterns`, `django-patterns`, `python-testing` | `python-reviewer`, `django-reviewer`, `django-build-resolver` |
   | Go | `Stack_specific/golang/` | `golang-patterns`, `golang-testing` | `go-reviewer`, `go-build-resolver` |
   | Rust | `Stack_specific/rust/` | `rust-patterns`, `rust-testing` | `rust-reviewer`, `rust-build-resolver` |
   | Swift | `Stack_specific/swift/` | `swiftui-patterns`, `swift-protocol-di-testing` | `swift-reviewer`, `swift-build-resolver` |
   | Kotlin | `Stack_specific/kotlin/` | `kotlin-patterns`, `kotlin-testing` | `kotlin-reviewer`, `kotlin-build-resolver` |
   | Java | `Stack_specific/java/` | _(none)_ | `java-reviewer`, `java-build-resolver` |
   | C# | `Stack_specific/csharp/` | `csharp-testing` | `csharp-reviewer` |
   | C++ | `Stack_specific/cpp/` | `cpp-testing` | `cpp-reviewer`, `cpp-build-resolver` |
   | PHP | `Stack_specific/php/` | _(none)_ | `php-reviewer` |
   | Perl | `Stack_specific/perl/` | `perl-patterns`, `perl-testing` | _(none)_ |
   | Ruby | `Stack_specific/ruby/` | _(none)_ | _(none)_ |
   | Dart / Flutter | `Stack_specific/dart/` | `dart-flutter-patterns` | `flutter-reviewer`, `dart-build-resolver` |
   | F# | `Stack_specific/fsharp/` | `fsharp-testing` | `fsharp-reviewer` |
   | HarmonyOS / ArkTS | `Stack_specific/arkts/` | _(none)_ | _(none)_ |
   | General web / frontend | `Stack_specific/web/` + `Stack_specific/design/` | `frontend-patterns`, `vite-patterns`, `e2e-testing` | `typescript-build-resolver` |

2. Copy each matched ruleset directory (the "Ruleset directory" column) from
   `<storeRoot>/Rules/`
   into `.claude/rules/` in the project root, dropping the `Stack_specific/`
   marker so the result is one subfolder per language:
   `.claude/rules/<lang>/<topic>.md`. Keep the rule filenames as-is.
   Do NOT copy `Global/` rules — global rules are already installed system-wide in
   `~/.claude/rules/`. Reference the copied language rules from the project
   `CLAUDE.md` when the project needs them enforced. Skip any tech choice with no
   matching ruleset entry.

3. Copy each matched skill directory from
   `<storeRoot>/Skills/Stack_specific/`
   into `.claude/skills/` in the project root — preserving directory structure.
   Deduplicate: if a skill is matched by multiple tech choices, copy it once.
   Skip any skill with no matching entry in the store.

4. Copy each matched agent file from
   `<storeRoot>/Agents/Stack_specific/`
   into `.claude/agents/` in the project root.
   Deduplicate: if an agent is matched by multiple tech choices, copy it once.
   Skip any agent with no matching file in the store.

## Gate 3 — Design Foundation (UI projects only)

Establish the project's visual language once, with the user, and persist it. This gate is the
antidote to UI that gets "designed without being designed" — every later UI milestone derives
from its output instead of re-improvising.

**Conditional.** Run this gate only if Gate 2's stack includes a user-facing frontend. For
CLI, library, or API-only projects, skip it with a one-line note ("no user-facing UI — Design
Foundation gate skipped") and move to Gate 4.

**Deferrable.** Even for a UI project, the user may defer this gate — "start with the backend,
design when the first screen arrives" — rather than completing it at setup. Offer this
explicitly. Deferring is safe: `implement-milestone` creates the design foundation on the
first UI milestone if it is absent. Do not force a full UI design foundation ahead of backend
milestones (skeleton, auth, domain logic) that gain nothing from it. If deferred, record
"Design Foundation: deferred to first UI milestone" and move to Gate 4.

When run, the gate produces a signed-off design. Its rule dependencies (`design/design-quality.md`
and `design/design-principles.md`) are already provisioned by Gate 2 for every frontend stack.

1. **Interview.** Invoke the `grill-me` skill to draw out the user's design opinions,
   references, brand, tone, and hard constraints. Do not infer a visual direction from
   silence — the user has strong opinions; surface them.
2. **Design.** Invoke the `frontend-design` skill to produce a visual direction grounded in
   the PRD and the interview, meeting `design-quality.md` (aesthetics) and `design-principles.md`
   (craft floor).
3. **Render for review.** Produce a self-contained `docs/styleboard.html` — palette
   swatches, real type specimens in the chosen faces, buttons and inputs in their
   hover/focus/active states, the signature element, and one sample composition. The user
   opens it in a browser (the terminal cannot show UI).
4. **Sign-off review loop.** Present the styleboard with a one-paragraph rationale (the choices
   and the one risk taken). The user reacts; revise and re-present. Repeat until the user
   explicitly approves. This is a hard gate — do not proceed on assumption.
5. **Persist.** On approval, write `docs/DESIGN.md` from the scaffold in this
   skill's own folder, filling every placeholder from the approved direction. `docs/DESIGN.md` is
   the canonical text source of truth; `docs/styleboard.html` is its committed rendered
   companion. No application code is written here — code-level tokens are seeded by the first
   UI milestone, which lifts the `:root` block from `docs/DESIGN.md` into the real token layer.

Present the design foundation summary. Wait for confirmation.

## Gate 4 — Milestones

Produce an ordered list of milestones, each with:
- Name
- Scope: what is included
- Done-criteria: how to confirm completion

Create docs/PROGRESS.md with this structure:

```markdown
# PROGRESS — [Project Name]

> Last updated: [date]

## Delivery Milestones

| Milestone | Scope | Done-criteria | Status |
|---|---|---|---|
| [name] | [scope] | [criteria] | pending |
```

One row per milestone. wrap-it-up fills in all other docs/PROGRESS.md sections on the first wrap.

Present the milestone plan. Wait for confirmation.

## Gate 5 — Git Workflow

Two things must be settled here, neither set as a silent default. Ask them as two separate
turns — one decision per message (see the global interaction-design doctrine), never
batched into a single prompt:

1. First turn — ask: "How will you handle git for this project?" Recommend feature branches
   with --no-ff merges into main. Wait for the answer.
2. Next turn — ask: "Is this project remote-backed or local-only? If remote-backed, what is
   the remote name and URL?" Wait for the answer.

Record `remote` to reflect **reality**, never a fabricated default — a recorded remote that
does not exist is what turns `wrap-it-up`'s push into a silent no-op:

- **Local-only** → set `"remote": null` and steer to a no-push workflow (`commit-only`, or
  `merge-to-main` without push).
- **Remote-backed** → verify or create it: `git remote add <name> <url>` if not already
  present, confirm with `git remote get-url <name>`, and record **only a name that
  resolves**. If the user intends a remote but has no URL yet, record `null` and note it as
  pending — do not assert `origin`.

Record the confirmed choices in .claude/wrap-it-up.json:

```json
{
  "gitWorkflow": "merge-to-main",
  "mainBranch": "main",
  "remote": "<remote name that resolves, or null for local-only>",
  "storeRoot": "<absolute path to the Lindholm Code store, from Gate 2>"
}
```

`remote` must name a remote that resolves via `git remote get-url`, or be `null` for a
local-only project. Never a name that does not exist. Adjust all values to match the
confirmed choices. Carry `storeRoot` over from the path resolved in Gate 2. Wait for
confirmation.

## Files Created

After all gates are confirmed, create:

**Directory skeleton** — create the three canonical folders (see Directory Layout):
- `product/` — starts empty; seed `product/.gitkeep` so the folder is tracked until the
  first milestone writes code (remove it naturally once code lands).
- `docs/` — already populated by the gates (PRD, PROGRESS, DESIGN, adr/).
- `resources/` — create it with a `resources/README.md` containing one paragraph:
  "User-supplied input material — starting documents, design references, inspiration,
  spreadsheets, and reference files you add by hand. Not generated by Claude.
  docs/DESIGN.md is the authored design foundation (output of the Design Foundation gate);
  raw inspiration and reference material the user brings in belongs here."

**.gitignore** (root) — write exactly these entries:
```
# Environment — matches anywhere in the repo (root, product/, subdirectories)
.env*

# Claude harness
.claude/

# User-supplied input (not committed)
resources/

# Graphify
.graphify_*
graphify-out/

# macOS
.DS_Store
.DS_Store?
._*
.Spotlight-V100
.Trashes

# Windows
Thumbs.db
ehthumbs.db
desktop.ini
```
The `docs/adr/` folder is committed to git, not ignored.

**docs/DESIGN.md** (UI projects only) — written at Gate 3 from the `DESIGN.md` scaffold in this
skill's own folder, alongside its rendered companion `docs/styleboard.html`. If Gate 3 was
deferred, both are created by the first UI milestone instead. Omitted entirely for non-UI
projects.

**CLAUDE.md** (root — stays at the repository root so the harness auto-loads it) — create it
from the `project-CLAUDE.md` scaffold in this skill's own folder. Fill every placeholder from
the confirmed gate decisions (Purpose from Gate 1; Tech Stack and Architecture from Gate 2;
visual language from Gate 3 if run; Git Conventions from Gate 5), fill the Project Layout
section from the Directory Layout above, reference any language-specific rulesets provisioned
in Gate 2 step 2 (including the `design/` rules for frontend stacks), and delete the scaffold's
guidance comments. Keep it project-specific: do not restate global preferences or the general
ruleset.

## Closing

After files are created, print a concise summary of all decisions made across the gates.
Then recommend `/implement-milestone` to begin Milestone 1 — the first step once the project is set up.
