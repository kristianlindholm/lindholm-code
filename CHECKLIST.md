# Checklist — build backlog

Open work items for this store. `MIGRATION.md` gates installation on these being closed.
Store-only: never installed into `~/.claude/`.

Claims below are tagged with their source: `[verified]` (read or measured directly in this
repo or `~/.claude/`), `[inference]` (reasoning from verified facts), `[reported]` (from a
subagent, not independently confirmed).

---

## 1. Command wrappers are obsolete and defeat `disable-model-invocation` — RESOLVED 2026-08-14

Severity: HIGH (the guard defeat) and LOW (the documentation drift). One root cause, two
consequences. Found 2026-08-13.

**Resolution.** All 13 wrappers retired to `Archive/Commands/Global/` with `git mv`; reason
recorded in `Archive/README.md`. `create-skill` Step 5 rewritten from "Add the slash command" to
"Verify how it is invoked"; root `CLAUDE.md`, `README.md` and `MIGRATION.md` updated to the
merged world. `add-to-vault`'s stale pointer to recording the vault path "in the command wrapper"
now names `$OBSIDIAN_CLAUDE_VAULT` instead.

**The open question below was settled against the official docs, with one correction to what
this entry originally claimed.** The docs state the opposite of the inference recorded here:
"if a skill and a command share the same name, the skill takes precedence... the command is never
consulted". That holds for the **user-typed** path. It does **not** hold for model invocation of
a skill barred by its own flag — confirmed from a live transcript, where the model called
`Skill(skill="implement-milestone")` against a guarded skill and got
`{"success":true,"commandName":"implement-milestone"}` with the wrapper's body injected. The
skill is removed from the model's registry by its own guard, and the unguarded wrapper is simply
what remains under that name. Worth reporting upstream via `/feedback`.

The docs did confirm the safe half outright: "A file at `.claude/commands/deploy.md` and a skill
at `.claude/skills/deploy/SKILL.md` both create `/deploy`", and `disable-model-invocation: true`
leaves the user able to type `/<name>` (menu visibility is governed by the separate
`user-invocable` flag, which no skill here sets). So retiring the wrappers costs no user-facing
entry point.

Original finding follows, for provenance.

### Background — what changed upstream

Claude Code merged slash commands and skills into a single concept:

- "Merged slash commands and skills, simplifying the mental model with no change in
  behavior" — `~/.claude/cache/changelog.md:3845` `[verified]`
- "Improved skills from `/skills/` directories to be visible in the slash command menu by
  default (opt-out with `user-invocable: false` in frontmatter)" —
  `~/.claude/cache/changelog.md:3976` `[verified]`

Before the merge, a skill could not be typed. A command wrapper existed solely to give the
user an entry point: typing `/implement-milestone` sent "Invoke the implement-milestone skill
and follow it exactly." That is the world this store was built for, and it is why
`create-skill` Step 5 instructs authors to ship a wrapper for every user-facing skill.

After the merge, a skill registers its own slash entry. `/implement-milestone` works with no
wrapper file present. The wrapper is no longer the way in.

Note the opt-out flag in the changelog is `user-invocable: false`, which is a *different*
control from `disable-model-invocation: true`. They govern opposite directions — the former
hides a skill from the user's slash menu, the latter stops Claude firing it from context.
This store uses `disable-model-invocation: true`, which remains the correct flag for its
intent. `[verified]`

### The collision

This store ships 13 command wrappers in `Commands/Global/`. Every one of the 13 shares its
name with an installed skill — verified by direct comparison of `Commands/Global/*.md`
against `Skills/Global/*/SKILL.md`. `[verified]`

Of those 13, **11 shadow a skill that carries `disable-model-invocation: true`**:
`add-to-vault`, `continue-project`, `create-skill`, `give-feedback`, `implement-milestone`,
`implement-task`, `new-project`, `save-for-later`, `save-session`, `security-check`,
`wrap-it-up`. The remaining two (`frontend-design`, `grill-me`) are intentionally
model-invocable and carry no guard. `[verified]`

Since the merge, skill and command entries share one registry. When both claim a name, one
wins. `[inference]`

### Evidence that the wrapper wins

Three files, read directly:

| Source | `description` |
|---|---|
| `~/.claude/skills/implement-milestone/SKILL.md` | "Execute the active milestone to best practice — scope what applies, then research, plan, build test-first, and review, stopping at the commit gate." (plus `disable-model-invocation: true`) |
| `~/.claude/commands/implement-milestone.md` | "Execute the active milestone to best practice, stopping at the commit gate." |
| The model-invocable skill listing in a live session | "Execute the active milestone to best practice, stopping at the commit gate." |

The live listing matches the **command** verbatim, not the guarded skill. `[verified]`

The obvious alternative explanation — that the installed skill has simply drifted to a
shorter description — was checked and ruled out: the installed skill file carries the long
description *and* the flag. `[verified]` The same mismatch appears on `wrap-it-up`.
`[verified]`

Conclusion: the entry offered for model invocation is the wrapper, which carries no guard, so
the guard on the skill file is never consulted. `[inference]`

### Evidence it happens in practice

A grep of `~/.claude/projects/` for model-initiated `Skill` tool calls targeting guarded
skills (`implement-milestone`, `implement-task`, `wrap-it-up`, `continue-project`,
`new-project`) returns **20 occurrences across 16 transcripts**. `[verified]`

These are skills the store deliberately marked as user-started only. `implement-milestone`
writes product code. `new-project` scaffolds an entire repository. `wrap-it-up` commits,
merges into `main`, and pushes. The flag states the intent; the wrapper defeats it.

### Consequence A — HIGH: the guard does not hold

`disable-model-invocation: true` is present, correct, and inert on the 11 skills that carry
it. Claude can start any of them autonomously by resolving to the wrapper.

Not CRITICAL — no data loss, no secret exposure, and every git action still passes through
its own confirmation gate. But it is a real defect on the store's most consequential skills,
and the artifact currently asserts a protection it does not deliver.

### Consequence B — LOW: the wrappers no longer do the documented job

The wrappers are not dead; they have been **repurposed**. They no longer serve the
user-typed path (the skill shadows them there) and instead function as the model-invocation
entry. Same root cause as A, viewed from the other side.

Documentation now describes a mechanism that no longer operates as written:

- `README.md:60` calls the 13 wrappers "the user-facing entry points to the lifecycle
  skills". That is the one thing they no longer are. `[reported]`
- `MIGRATION.md:52` asks the installer to verify "Each `/command` appears in the slash menu
  and loads its skill". The check passes, but because the *skill* self-registered — not
  because the wrapper functioned. `[inference]`
- `Commands/Global/add-to-vault.md` is the only wrapper carrying content beyond the
  boilerplate line: a paragraph instructing the user to set `$OBSIDIAN_CLAUDE_VAULT` before
  running. On the typed path that paragraph is never delivered. This is real instruction
  loss, not just redundancy. `[reported]`
- `create-skill` Step 5 still instructs authors to ship a wrapper for every user-facing
  skill, propagating the obsolete pattern to future artifacts. `[verified]`

### Measurement caveat

An earlier subagent reported the wrapper body was delivered "0 times in 216 invocations".
That is overstated. Direct measurement for `implement-milestone` found roughly 8 genuine
wrapper-body deliveries against 37+ typed invocations (excluding this investigation's own
transcripts, which quote the file). The direction holds — typed entry overwhelmingly reaches
the skill directly — but the wrapper path is not wholly unused. `[verified]`

The same subagent reported "11 of 13" wrappers collide with a skill name. The correct figure
is 13 of 13 colliding; 11 of those shadow a *guarded* skill. `[verified]`

Neither correction affects the HIGH, which rests on the description mismatch and the 20
observed calls.

### Files affected

- `Commands/Global/` — all 13 wrappers
- `Skills/Global/create-skill/SKILL.md` — Step 5, the authoring instruction
- `CLAUDE.md` (repo root) — the "Commands are not a fourth artifact type" paragraph
- `README.md` — the wrapper description and repo-layout section
- `MIGRATION.md` — the Commands install step (line 24) and the verify step (line 52)
- `~/.claude/commands/` — the installed copies

### Proposed remedy — decide between two options

**Option 1 (recommended): retire the wrappers to `Archive/`.** Post-merge they are redundant on
the path they were written for, and harmful on the path they now serve. Removing the unguarded
entry restores the guard.

Retire, never delete — per this store's `CLAUDE.md`: `git mv` each file to
`Archive/Commands/Global/<name>.md` so history follows it and the path mirrors the original,
making restoration a straight move back. Record the reason in `Archive/README.md`. This applies
to the installed copies in `~/.claude/commands/` too: back them up rather than removing them
outright.

**Option 2: keep the wrappers and guard them.** Add `disable-model-invocation: true` to the
11 wrappers shadowing guarded skills. Closes the HIGH but leaves 13 redundant files and the
inaccurate documentation in place.

Either way the documentation must be updated to the merged world.

### Execution notes if Option 1 is taken

1. Confirm first, empirically, that `/name` still resolves with the wrapper absent — delete
   one wrapper (`save-for-later` is the lowest-risk) and verify the slash entry still works
   before removing the rest. Do not bulk-delete on the strength of the changelog alone.
2. Relocate `add-to-vault`'s `$OBSIDIAN_CLAUDE_VAULT` paragraph into
   `Skills/Global/add-to-vault/SKILL.md` before deleting that wrapper, or the instruction is
   lost outright.
3. Rewrite `create-skill` Step 5 — from "ship a wrapper when a user would invoke the skill
   directly" to how user-invocability is actually controlled after the merge. Confirm the
   current flag semantics before writing, rather than assuming.
4. Update root `CLAUDE.md`, `README.md`, and `MIGRATION.md` (steps 24 and 52) to match.
5. Remove the installed copies from `~/.claude/commands/`. Back up first per
   `MIGRATION.md:16`.
6. Re-verify that all 11 guarded skills refuse model invocation afterwards, by attempting one.

### Open question

The precise resolution rule when a skill and a command claim the same name is inferred from
observed behaviour, not from documentation. Worth confirming against current Claude Code docs
before acting, since the remedy depends on it.

---

## 2. `security-check` never invokes `security-reviewer` — RESOLVED 2026-08-13

Severity: MEDIUM. Found 2026-08-13, incidentally while investigating item 1.

**Resolution.** The two were split along the scope line rather than merged. `security-reviewer`
gained an explicit scope contract (diff-scoped by default, codebase-scoped only for
`/security-check`); `implement-milestone` and `implement-task` now state the diff scope when
dispatching; `security-check` dispatches the agent codebase-scoped for the code-level areas and
keeps privacy, git-history secrets, severity and the verdict; `security.md` step 5 points at
`/security-check`.

A codebase-wide audit was also added as a **Final gate** checkbox in `docs/PROGRESS.md` — not a
milestone, not run by `/implement-milestone`, not triggered by `wrap-it-up`. `continue-project`
surfaces it at session start once every milestone is complete; `/security-check` ticks it.
`new-project` seeds it, and `wrap-it-up`'s PROGRESS template was corrected to preserve it (that
template was also missing the Delivery Milestones table — a pre-existing gap, now closed).

Original finding follows, for provenance.

`Rules/Global/security.md` states a Security Response Protocol: "If security issue found: 1.
STOP immediately. 2. Use **security-reviewer** agent." `[verified]`

`Skills/Global/security-check/SKILL.md` performs its entire audit inline and never references
`security-reviewer`, or any agent, anywhere in its 68 lines. `[verified]`

So the store ships a dedicated security-audit agent and a dedicated security-audit skill that
do not know about each other. Either the skill should delegate to the agent at its findings
step, or `security.md`'s protocol should acknowledge that `/security-check` is the inline
route and the agent is for the ad-hoc case. Decide which, then make the two agree.

Unrelated to item 1 beyond having surfaced during the same investigation.

---

## 3. Two stack artifacts tell Claude to trigger `/security-check` itself — RESOLVED 2026-08-13

Severity: MEDIUM. Found 2026-08-13 while resolving item 2.

**Resolution.** Both were repointed at the diff-scoped `security-reviewer` agent, which is what
they actually described. `nuxt/security.md` now treats those routes as a `code-review.md`
security trigger requiring `security-reviewer` at the milestone review step, scoped to the
route's diff, and states explicitly that `/security-check` is not a per-route tool.
`backend-patterns` now has `security-reviewer` cover abuse cases at the review step, scoped to
the endpoints in hand. A third reference — `angular-developer/SKILL.md:152` — was checked and
left alone: it is a "Related Skills" pointer, not a trigger instruction.

Note this removed the contradictory *instructions* but not the *mechanism*: item 1's unguarded
wrapper still makes `security-check` model-invocable. Item 1 remains open.

Original finding follows, for provenance.

`Skills/Global/security-check/SKILL.md` carries `disable-model-invocation: true` — by design,
only the user starts it. Two stack artifacts contradict that: `[verified]`

- `Rules/Stack_specific/nuxt/security.md:42` — "**Auto-trigger** `/security-check` only for
  routes that make external network requests (server `$fetch`), handle auth tokens or
  credentials, or perform sensitive mutations or authorization checks."
- `Skills/Stack_specific/backend-patterns/SKILL.md:438` — "design the HTTP contract deliberately
  and use the `security-check` skill".

This mattered less when the skill audited inline. After item 2's resolution `/security-check`
dispatches a codebase-wide agent audit, so an automatic trigger is materially more expensive —
and on a Nuxt project it could fire on any sensitive route change.

Compounding it, the unguarded command wrapper from item 1 makes `security-check` genuinely
model-invocable today, so these instructions are not merely aspirational.

Decide the intent: either those two artifacts should point at the diff-scoped
`security-reviewer` agent instead (most likely correct — they describe per-route and per-endpoint
concerns, not whole-codebase audits), or the skill's guard is wrong. Fixing item 1 removes the
mechanism but not the contradictory instructions.
