# Framework Migration Guide

How to install this framework into the main Claude Code folders once the work is
done. Global artifacts install once into `~/.claude/`; language-specific artifacts
are never installed globally — `/new-project` provisions them per project.

Paths use `~/.claude/` throughout. On Windows that is `C:\Users\<you>\.claude\`.

## Before you start

- [ ] The global developer-preferences `CLAUDE.md` deliverable is final (`Claude_MD/global-CLAUDE.md`).
- [ ] Every user-invoked skill has been run via its `/command` and behaves as written.
- [ ] Every cross-reference resolves — no broken links to a skill, agent, rule, or command.
- [ ] No project-specific or "Lindholm" wording remains in any Global artifact.
- [ ] `CHECKLIST.md` items are closed.
- [ ] The current `~/.claude/` is backed up — copy `skills/`, `commands/`, `agents/`, `rules/`, and `CLAUDE.md` somewhere safe. Migration overwrites by name.

## Install global artifacts (drop the `Global/` wrapper)

The `Global/` folder name is an authoring marker. It is NOT kept at the destination — files flatten one level up.

- [ ] Skills — copy each `Skills/Global/<name>/` folder in full to `~/.claude/skills/<name>/`.
  - Copy the whole folder, not just `SKILL.md`. Disclosed reference files travel with it — e.g. `create-skill` ships `writing-great-skills.md` and `GLOSSARY.md`; `codebase-design` ships `DEEPENING.md` and `DESIGN-IT-TWICE.md`; `tdd` ships `tests.md`, `mocking.md`, and `refactoring.md`.
- [ ] Commands — copy each `Commands/Global/<name>.md` to `~/.claude/commands/<name>.md`.
- [ ] Agents — copy each `Agents/Global/<name>.md` to `~/.claude/agents/<name>.md`.
- [ ] Rules — copy each `Rules/Global/<topic>.md` to `~/.claude/rules/<topic>.md`.

## Install the global CLAUDE.md

- [ ] Copy `Claude_MD/global-CLAUDE.md` to `~/.claude/CLAUDE.md` (rename to `CLAUDE.md`).
  - This replaces your current global file. It is NOT this repo's project `CLAUDE.md`, which governs how the framework is built and is never migrated.
  - The per-project scaffold lives with its skill at `Skills/Global/new-project/project-CLAUDE.md` and travels with that skill when it installs; it is not a separate migration step.

## Language-specific artifacts — do NOT install globally

- [ ] Leave `Skills/Stack_specific/`, `Agents/Stack_specific/`, and `Rules/Stack_specific/` in the store.
  - `/new-project` provisions these per project, scoped to that project's stack.
  - Language-specific rules land at `<project>/.claude/rules/<lang>/<topic>.md` (one subfolder per language) and are linked from the project `CLAUDE.md` when needed.
  - Global artifacts are assumed already installed and are not re-copied per project.

## Store-only files — never installed

- [ ] Leave these in the store. They govern how the framework is built and have no place in `~/.claude/`:
  - `CLAUDE.md` — this repo's governing document, not the global preferences file.
  - `CHECKLIST.md` — the build backlog.
  - `COVERAGE.md` — the language-support matrix and gap policy.
  - `MIGRATION.md` — this guide.
- The `Claude_MD/` folder is a staging area: its `global-CLAUDE.md` installs as `~/.claude/CLAUDE.md` (see above); the folder itself is not copied.

## Verify after install

- [ ] Each `/command` appears in the slash menu and loads its skill (spot-check `/create-skill`, `/wrap-it-up`).
- [ ] Each user-invoked skill loads, and its disclosed references resolve (open `create-skill` — its `writing-great-skills.md` and `GLOSSARY.md` links work).
- [ ] The model-invoked skills (`accessibility`, `codebase-design`, `tdd`) still carry their descriptions and fire from context.
- [ ] Agents are listed and callable.
- [ ] Global rules are present under `~/.claude/rules/`.
- [ ] `~/.claude/CLAUDE.md` reads correctly and contradicts none of the installed rules.

## Conflicts and watch-outs

- Name collisions overwrite. Any existing `~/.claude/` skill, command, or agent with the same name is replaced — resolve each intentionally (this is why you backed up).
- This store bundles `writing-great-skills` as a reference inside `create-skill/`, not as a standalone skill. If you rely on a standalone `/writing-great-skills`, this framework does not reproduce it — keep or remove the existing one deliberately.
- `settings.json` is not part of this store. If your workflow depends on settings (attribution, permissions, hooks), migrate or re-author it separately.

## Rollback

- [ ] If anything misbehaves, restore the backed-up `~/.claude/` folders.
