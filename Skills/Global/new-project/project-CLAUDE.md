# [Project Name]

<!-- Scaffold used by /new-project. Replace every [bracketed] placeholder and delete
     these comment blocks. Keep this file project-specific: global preferences and the
     general ruleset live in ~/.claude/CLAUDE.md and ~/.claude/rules/ and must not be
     restated here.

     THIS FILE IS AN INDEX, NOT A CONTAINER. It is loaded into every session, so it holds
     what binds EVERY change plus one line per topic pointing at the file holding the rest.
     Detail belongs in docs/rules/<practice>.md, docs/architecture/<subsystem>.md and
     docs/adr/. See coding-style.md, "Authored governance documents". -->

## Purpose
[One sentence: what this project does and for whom. From Gate 1.]

## Project Layout
<!-- Fixed layout enforced by /new-project. Keep this section — every session reads it to
     know where things live. -->
- `product/` — the actual product code. All source, dependencies, and build/test commands
  live here (run them from `product/`).
- `docs/` — authored governing documents: `PRD.md`, `PROGRESS.md`, this project's own rules in
  `docs/rules/`, subsystem detail in `docs/architecture/`, ADRs in `docs/adr/` (indexed by its
  `README.md`), and for UI projects `DESIGN.md` plus its rendered companion
  `docs/styleboard.html`.
- `resources/` — user-supplied input material (starting documents, inspiration, spreadsheets).
- Root holds only `CLAUDE.md`, `.gitignore`, and the `.claude/` config, which the harness
  requires at the repository root.

## Tech Stack
[Each technology and the reason it was chosen. From Gate 2.]

## Architecture
[2-3 sentences describing the structural shape of the system and its key boundaries.
From Gate 2. This is the whole section — the shape, not a tour of the modules. How one
subsystem is built, and why it is built that way, goes in docs/architecture/<subsystem>.md
and gets a line in the Rules index below.]

## Git Conventions
[GitHub-backed or no version control (from Gate 6 — Version Control). For a GitHub-backed
project: the remote (normally `origin`), branch and merge strategy (default: feature
branches with `--no-ff` merges into main), and commit message format. For a project with
no version control, state "No version control." No local-only git repositories.]

## Design Foundation
[UI projects only. docs/DESIGN.md is the signed-off source of truth for the visual language;
docs/styleboard.html is its rendered companion. Every UI milestone derives from docs/DESIGN.md
and must not re-decide it. Remove this section for non-UI projects, or if the Design Foundation
gate was deferred until the first UI milestone.]

## Cross-cutting rules
[Only what binds EVERY change to this project — the invariants a reader must carry into any
file they touch. Typically a handful. A rule that binds one practice or one subsystem is not
cross-cutting; it goes to its topic file and is reached through the index below.

Placement decides how often a rule is in force, so when in doubt leave it here. This file is
loaded into every session; a topic file is opened only when the work selects that practice, so
a rule filed there binds a milestone that touches it and binds nothing else. That is the point
for a testing rule, and a quiet failure for an invariant that should have stayed here. In a
conflict the cross-cutting rule wins.

State the rule, not its history. If it needs paragraphs to justify itself, those paragraphs
are an ADR and the rule points at it.]

## Rules
<!-- The index. One line per topic: the file, then what it covers. Consult the relevant file
     rather than reproducing its content — the same arrangement ~/.claude/CLAUDE.md uses over
     ~/.claude/rules/. Covers both trees, because a reader does not care which one a rule came
     from: .claude/rules/<stack>/ holds rules provisioned from the store and is never hand-
     edited; docs/rules/ holds this project's own. Where both name the same topic, read both: the
     project file overrides on the points it addresses and the stack file governs the rest. It
     replaces nothing.

     A ROW EXISTS ONLY ONCE ITS FILE DOES. At creation that means the stack rules provisioned
     in Gate 2 and nothing else: docs/rules/, docs/architecture/ and docs/adr/ all start empty
     and gain their first row when the first rule, subsystem note or ADR is agreed. A row
     pointing at a file that does not exist is a promise that breaks on first use, and the
     wrap truth pass will report it.

     Replace the example rows below with one per provisioned stack rule, then delete this
     comment. -->

| File | Covers |
|---|---|
| `.claude/rules/<stack>/<topic>.md` | [what this provisioned rule covers] |
