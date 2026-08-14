# Archive

Retired artifacts, kept for provenance and possible restoration.

## The rule

Nothing under `Archive/` is ever installed into the global `~/.claude/` setup or provisioned
into a project. It is outside every install path described in the root `CLAUDE.md` under
"How this store is used", and outside the coverage parity requirement in `COVERAGE.md`.

Paths mirror the artifact's original location, so a restoration is a straight move back:
`Archive/Skills/Global/accessibility/` came from `Skills/Global/accessibility/`.

Move with `git mv` so history follows the file.

An archived artifact is not maintained. Before restoring one, read its entry below — known
defects are recorded there precisely so a restore does not silently reinstate them.

## Entries

### `Skills/Global/accessibility/` — retired 2026-08-11

**What it was.** A model-invoked skill implementing WCAG 2.2 Level AA: a five-step POUR
procedure, a cross-platform ARIA / SwiftUI / Compose mapping table, three code examples,
four anti-patterns, and a seven-item checklist.

**Why it was retired.** Roughly 80 per cent of its content was assistive-technology plumbing
— ARIA attributes, the accessibility tree, screen reader traits, live regions, iOS and Android
accessibility traits. This store serves internal tooling operated by Emendo consultants, in
house and when presenting at client sites. Nothing is delivered to client staff and nothing is
sold as a product, so no conformance regime applies and the skill had no audience. It also
auto-fired on any UI-shaped prompt, because it carried no `disable-model-invocation` flag.

Interaction quality that the skill had been carrying incidentally is now handled properly
rather than lost: usage context is captured at `new-project` Gate 1, and the interaction
decisions (keyboard paths for frequent actions, focus behaviour on state change, control sizing
by frequency, presentation robustness) are asked at `implement-milestone` Step 5 and recorded in
the project's `DESIGN.md`. See the note in `COVERAGE.md`.

**Known defects — do not reinstate these if the skill is ever restored.**

- L40 cites SC 2.4.11 for a visible focus indicator. Visible focus is **SC 2.4.7 (AA)**;
  2.4.11 is Focus Not Obscured (Minimum) (AA); the indicator-geometry criterion is
  **2.4.13 Focus Appearance, Level AAA**.
- L14 presents Focus Appearance as part of the AA target. It is AAA.
- L39 and L123 state a 24x24 CSS pixel target size without SC 2.5.8's five exceptions
  (spacing, equivalent, inline, user agent control, essential).
- L123 lists 44x44pt as a target-size figure without marking it as platform guidance
  (Apple HIG) rather than WCAG. The store's own `design-principles.md` separately cited
  44x44px against SC 2.5.8, which is wrong — 44x44 is SC 2.5.5, Level AAA.
- L33 states the contrast thresholds without SC 1.4.3 and 1.4.11's exceptions for inactive
  and disabled components, logotypes, and incidental or decorative text.
- L35 states reflow as "400% zoom" rather than SC 1.4.10's 320 CSS pixel equivalent, and omits
  the exception for content requiring two-dimensional layout (data tables, maps, complex charts).
- L139-142 cross-references `frontend-patterns` and `swiftui-patterns` as related skills.
  Neither exists as a Global skill; both are `Stack_specific`.
- Two genuine criteria were missing: SC 3.3.8 Accessible Authentication (Minimum) (AA) and
  SC 3.2.6 Consistent Help (A).

### `Commands/Global/` — all 13 wrappers, retired 2026-08-14

**What it was.** One four-line wrapper per user-invoked Global skill: frontmatter `description`
plus the body line `Invoke the <name> skill and follow it exactly.` They existed to give the user
a `/name` entry point, back when a skill could not be typed directly.

**Why it was retired.** Claude Code merged slash commands and skills. The official documentation
states that "A file at `.claude/commands/deploy.md` and a skill at `.claude/skills/deploy/SKILL.md`
both create `/deploy`", and the changelog records the merge as "simplifying the mental model with
no change in behavior". A skill now registers its own slash entry, so every wrapper was redundant
on the path it was written for.

Worse, each wrapper was a second registry entry under the same name carrying no
`disable-model-invocation` flag. For the 11 wrappers shadowing a guarded skill this defeated that
guard: the skill was correctly barred from the model's registry by its own flag, and the unguarded
wrapper filled the slot. Confirmed in a live transcript — the model called
`Skill(skill="implement-milestone")` against a guarded skill and received
`{"success":true,"commandName":"implement-milestone"}`, with the wrapper's body injected rather
than the skill's. The documented precedence rule ("the skill takes precedence... the command is
never consulted") holds for the user-typed path but not for model invocation of a barred skill.

**Before restoring any of these.** Do not restore a wrapper whose skill carries
`disable-model-invocation: true` — it reopens the bypass. If a wrapper is ever genuinely needed
again, give it the same guard as its skill.

**One content note.** `add-to-vault.md` was the only wrapper carrying anything beyond the
boilerplate: a paragraph on setting `$OBSIDIAN_CLAUDE_VAULT` before running. That ground is
already covered by the skill's Steps 2 and 3, and the skill's stale pointer to "record it in the
command wrapper" was rewritten to name the environment variable instead. Nothing was lost.
