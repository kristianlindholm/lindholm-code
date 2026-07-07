---
name: create-skill
description: Create a new skill — justify, locate, choose invocation, write, add a command, and test.
disable-model-invocation: true
---

# Create Skill

Produces a new, immediately usable skill: a `SKILL.md` (plus any disclosed reference) and a slash command that invokes it, meeting the conventions and observed to work.

This file is the procedure — the ordered actions. For how to write the skill _well_ — information hierarchy, leading words, progressive disclosure, pruning — invoke [`writing-great-skills`](writing-great-skills.md) at Step 4 and apply it; its vocabulary is defined in [`GLOSSARY.md`](GLOSSARY.md). Point to that reference, never restate it here.

## Step 1 — Grill, then justify the skill

Before evaluating anything, invoke the `grill-me` skill to interview the user until the skill's
intent is concrete: what it produces, when it should fire, the failure mode it prevents, and how
it differs from existing skills. Do not infer the skill from a one-line prompt — grill until the
three tests below can be answered from evidence rather than guessed.

A skill is warranted only when all three hold:

- The pattern recurs across multiple sessions or projects.
- Claude would handle it differently without explicit guidance.
- No existing skill in the library already covers it — search the library before writing.

Done: the interview has made the intent concrete, and each of the three is stated and holds. If any fails, stop — handle it inline or as a rule in `CLAUDE.md`, and do not create the skill.

## Step 2 — Locate the skill

Place the skill where Claude Code loads it:

- `~/.claude/skills/<name>/SKILL.md` — the default. A user-level skill is available in every project.
- `./.claude/skills/<name>/SKILL.md` — only when the skill is specific to the current project.

`<name>` is kebab-case and matches the folder, the `name` frontmatter, and the H1 title.

Done: the folder exists at the chosen scope, and its name equals the intended `name` and H1.

## Step 3 — Choose invocation

- **User-invoked** (default) — fires only when the user types its name. Frontmatter: `disable-model-invocation: true`, and `description` (if present) is a one-line human-facing summary. Choose when the skill needs deliberate intent or model detection would be unreliable.
- **Model-invoked** — Claude fires it from context, and other skills can reach it. Frontmatter: keep a `description` written as trigger conditions ("Use when…"), never a workflow summary.

When uncertain, choose user-invoked: a wrongly-firing model-invoked skill is noise; a user-invoked skill that fires slightly less often costs nothing. See the Invocation section of [`writing-great-skills`](writing-great-skills.md) for the full context-load / cognitive-load trade-off.

Done: invocation type chosen, frontmatter written to match, and the reason it beats the alternative stated.

## Step 4 — Write the skill

Invoke [`writing-great-skills`](writing-great-skills.md) and apply it. On top of that reference, these conventions apply:

- H1 title matching the folder and `name`, then one sentence stating what the skill produces.
- Any hard constraint (do not implement, do not skip, do not begin new work) on the line immediately after that sentence.
- Numbered steps, each with a bold title and a checkable completion criterion — what exists when the step is done.
- English only. No emojis or icons. A list wherever a list carries the meaning as well as prose.

Done: `SKILL.md` exists, every step ends on a completion criterion, and nothing duplicates the reference or its glossary.

## Step 5 — Add the slash command

Add a command only when a user would invoke this skill directly. A pure sub-skill that only other skills invoke needs none — skip to Step 6. When a user would type it, a matching command makes it invokable inline. Create one at the same scope as the skill:

- `~/.claude/commands/<name>.md` for a user-level skill, or
- `./.claude/commands/<name>.md` for a project skill.

Its body invokes the skill, so typing `/<name>` triggers it:

```
---
description: <one line — what typing /name does>
---
Invoke the <name> skill and follow it exactly.
```

Done: either a user would type this skill and `/<name>` loads and runs it, or it is a pure sub-skill and no command was created.

## Step 6 — Test the skill

An untested skill may not produce the intended behaviour. Testing a skill is test-driven development applied to documentation: confirm the failure exists before trusting the guidance, then confirm the guidance removes it. Match the test to the skill type.

- **Technique or reference skill** — invoke it on a realistic scenario and watch Claude run it. Check: does it load, stop where it must, ask for confirmation where required, produce the stated output at each step, and resist its most likely failure mode? When behaviour deviates, find the ambiguous step, tighten its wording, and repeat.
- **Discipline skill** (one that enforces a rule against an incentive to skip it) — run the scenario in a fresh subagent _without_ the skill first and watch it fail, capturing the exact rationalizations it uses. That baseline proves the skill is needed and tells you what it must counter; if it reveals the skill targets the wrong failure, return to Step 4. Then run the same scenario under pressure _with_ the skill and confirm compliance, closing each new loophole. See [`testing-skills.md`](testing-skills.md) for the pressure-scenario and wording methodology.

A skill with no command — a pure sub-skill or one only Claude fires from context — is triggered by presenting the context its `description` should fire on, or by invoking it from the skill that depends on it, rather than by typing `/<name>`.

Done: the skill loads and runs on a realistic scenario without deviation; for a discipline skill, Claude complies under the pressure that made the baseline fail.
