# Lindholm Code — Claude Code Foundation

## Purpose
This repository is the authoritative source for the Claude Code configuration:
the agents, skills, and rules that govern how Claude works across every project.
It is built for professional software development and may be adopted by others.
Everything here must be correct, consistent, and self-explanatory without outside
context.

This file is the governing document for the repository: it defines the purpose and
the rules for building everything in it. It is not the global `~/.claude/CLAUDE.md`.
The shipped global developer-preferences `CLAUDE.md` is a separate deliverable that
will be authored during this project.

## What lives here
- `Agents/`  — delegated specialist executors that run in an isolated context
              (reviewers, build-resolvers, planner, architect).
- `Rules/`   — the standards Claude must meet.
- `Skills/`  — the procedures that tell Claude how to perform a task.
- `Claude_MD/` — the shipped global developer-preferences file
              (`global-CLAUDE.md`, installed to `~/.claude/CLAUDE.md`) and nothing
              else. Unlike the three folders above it is not split into
              `Global/`/`Stack_specific/`.

Each top-level folder is split into `Global/` and `Stack_specific/`. These
folder names are authoring markers that record where an artifact is installed —
they are not preserved at the destination:
- `Global/` artifacts are installed once into the global `~/.claude/` setup.
- `Stack_specific/` artifacts are provisioned per-project, scoped to the
  project's stack — its languages, frameworks, or infrastructure choices
  (Docker, a database, a deployment pipeline). The distinction is the install
  destination, not the topic: what every project should get lives in `Global/`;
  what only some stacks need lives in `Stack_specific/`.

## The core distinction (read before adding anything)
- Rules tell you WHAT standard to meet.
- Skills tell you HOW to do the work.
- Agents are WHO does delegated work in a separate context.

If a proposed artifact does not clearly fit exactly one of these, stop and
reconsider before creating it. Precedence: stack-specific overrides global;
user instructions override everything.

Commands are not a fourth artifact type: a skill ships a paired command wrapper in
`Commands/`, installed to `~/.claude/commands/` (see `create-skill` Step 5), only
when a user would invoke it directly. Invocation and the command are independent: the
command lets a user type `/<name>`; model-invocation (no `disable-model-invocation`)
lets Claude and other skills reach the skill from context. A skill may be both —
model-invocable *and* shipped with a command — so a skill invoked by other skills is
not barred from having one, but a pure sub-skill that a user never types needs none.

## Conventions for building everything

### Agents (`Agents/<scope>/<name>.md`)
Frontmatter schema — all four fields required:
- `name` — kebab-case, matches the file name.
- `description` — states WHEN to use the agent in explicit trigger language
  ("Use immediately after...", "MUST BE USED for...").
- `tools` — YAML array of the minimum tools the agent needs
  (e.g. `["Read", "Grep", "Glob"]`); nothing more.
- `model` — one of `opus`, `sonnet`, `haiku`.

Body: include the standard "Prompt Defense Baseline" section.

### Skills (`Skills/<scope>/<name>/SKILL.md`)
Frontmatter schema:
- `name` (required) — kebab-case; matches the folder name and the H1 title.
- `description` (required) — for model-invoked skills, written as trigger
  conditions ("Use when..."); for user-invoked skills, a one-line human-facing summary.
- `disable-model-invocation: true` (optional) — present only on skills that must be
  invoked explicitly by the user. Omit it on any skill that other skills invoke or
  that Claude should fire from context (e.g. `grill-me`). Ship a command wrapper only
  when a user would type the skill directly, independent of this flag.

Body: actionable and ordered — concrete numbered steps, each ending on a completion
criterion; not prose essays.

Naming patterns for `Stack_specific/` skills — pick the one that matches the scope:
- `<language>-patterns` / `<language>-testing` (e.g. `react-patterns`,
  `golang-testing`) — a pattern or testing library for one language or framework.
  This is the default.
- `<tool>-patterns` (e.g. `docker-patterns`, `redis-patterns`) — guidance for a
  tool or piece of infrastructure, language-agnostic, attached to a project by
  its tooling choice.
- `<framework>-developer` (e.g. `angular-developer`) — a comprehensive framework
  guide with a `references/` folder of disclosed deep-dive files. Reserve for
  frameworks large enough to need one.
- `<language>-<idiom>` (e.g. `swift-protocol-di-testing`) — a single idiom
  strongly tied to one language, when it does not fit the language's patterns or
  testing skill.

New skills use one of these four forms; a name outside them is a defect.

### Rules (`Rules/<scope>/<topic>.md`)
- One topic per file. State the standard, then a checklist or severity table.
- Cross-link related rules by relative path.
- Stack-specific rules state only what differs from or overrides Global.

### Universal
- English only, in code, comments, and content.
- No emojis or icons anywhere.
- Clean, modern, explicit over clever.
- `Stack_specific` (capital S, underscore) is the canonical folder name;
  any reference using another form — including the retired `Language_specific` —
  is a defect to fix.
- No ECC bleed. No artifact may reference "ECC" in its naming, frontmatter
  (`origin: ECC`), or content, and none may cross-reference ECC-only commands,
  agents, or skills that do not exist in this store. Remove on sight. Material
  adopted from ECC must be renamed and stripped of ECC references before it lands.

### Curation

- Adopt, don't bloat. Material adopted from other sources (ECC, third-party
  skills) is kept at full fidelity — the goal is fewer *unused* skills, not
  thinner content. Strip only what does not belong: foreign branding, dead
  references, and framework-specific code that lives elsewhere.
- Justify before adding. Before creating a skill, agent, or rule, justify it
  against the `create-skill` tests: it recurs across projects, it changes how
  Claude behaves, and nothing here already covers it. Prefer folding one
  valuable rule into an existing artifact over creating a thin new one.
- Keep framework-specific content in its stack artifact. Global skills stay
  stack-agnostic; framework code (test patterns, build configs, bundler
  rules) lives in the matching `Stack_specific` skill — never duplicated.
- References must resolve. Every cross-reference to a skill, agent, rule, or
  command must point to an artifact that exists in this store. Broken
  references are defects to fix on sight.

## Definition of done for the store
1. Coverage parity: each supported language has a consistent set
   (rules + patterns skill + testing skill + reviewer agent + build-resolver).
   Gaps are intentional, not accidental — recorded in `COVERAGE.md`, which is the
   source of truth for per-language coverage and the build-on-demand gap policy.
2. Structural consistency: naming, layout, and frontmatter uniform throughout.
3. Tooling integrity: meta-skills (`new-project`, `continue-project`,
   `create-skill`, `wrap-it-up`) reference only paths and names that exist.
4. Coherence: no rule contradicts another rule or a skill.
5. Activation correctness: every skill/agent triggers when intended, stays
   quiet otherwise.
6. The shipped global `~/.claude/CLAUDE.md` is authored and consistent with the
   rules in `Rules/Global/`.

## How this store is used
Global artifacts are installed once into `~/.claude/`, flattened (no `Global/`
folder at the destination):
- Rules: each file copied directly into `~/.claude/rules/<topic>.md`.
- Skills: each `<name>/` folder copied into `~/.claude/skills/<name>/` in full — `SKILL.md` plus any disclosed reference files it links to (e.g. `GLOSSARY.md`), which must travel with it.
- Agents: each file copied into `~/.claude/agents/<name>.md`.

Stack-specific artifacts are provisioned per-project by `new-project` into the
project's `.claude/` directory, scoped to the project's stack. Stack-specific
rules are copied into `<project>/.claude/rules/<lang>/<topic>.md` — one subfolder
per language, since every language reuses the same generic rule filenames
(`coding-style.md`, `testing.md`, ...) and a flat layout would overwrite them.
They are linked from the project-level `CLAUDE.md` when needed. Global artifacts
are assumed already installed and are not re-copied per project.

Keep every artifact copy-friendly: self-contained, no absolute-path assumptions
inside it.
