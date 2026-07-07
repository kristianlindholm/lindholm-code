# [Project Name]

<!-- Scaffold used by /new-project. Replace every [bracketed] placeholder and delete
     these comment blocks. Keep this file project-specific: global preferences and the
     general ruleset live in ~/.claude/CLAUDE.md and ~/.claude/rules/ and must not be
     restated here. -->

## Purpose
[One sentence: what this project does and for whom. From Gate 1.]

## Project Layout
<!-- Fixed layout enforced by /new-project. Keep this section — every session reads it to
     know where things live. -->
- `product/` — the actual product code. All source, dependencies, and build/test commands
  live here (run them from `product/`).
- `docs/` — authored governing documents: `PRD.md`, `PROGRESS.md`, ADRs in `docs/adr/`, and
  for UI projects `DESIGN.md` plus its rendered companion `docs/styleboard.html`.
- `resources/` — user-supplied input material (starting documents, inspiration, spreadsheets).
- Root holds only `CLAUDE.md`, `.gitignore`, and the `.claude/` config, which the harness
  requires at the repository root.

## Tech Stack
[Each technology and the reason it was chosen. From Gate 2.]

## Architecture
[2-3 sentences describing the structural shape of the system and its key boundaries.
From Gate 2.]

## Git Conventions
[Branch strategy, merge strategy, and commit message format. From Gate 4.]

## Design Foundation
[UI projects only. docs/DESIGN.md is the signed-off source of truth for the visual language;
docs/styleboard.html is its rendered companion. Every UI milestone derives from docs/DESIGN.md
and must not re-decide it. Remove this section for non-UI projects, or if the Design Foundation
gate was deferred until the first UI milestone.]

## Project Rules
[Project-specific instructions for Claude: patterns to follow, constraints to respect,
and decisions already made that must not be relitigated. Only what is true about THIS
project and extends or overrides the general rules.]

## Language Rules
[If language-specific rulesets were provisioned into .claude/rules/<lang>/, reference
the ones this project enforces, e.g. ".claude/rules/python/testing.md". Remove this
section if no language rulesets apply.]
