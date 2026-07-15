---
name: add-to-vault
description: Capture Claude's explanations from the current conversation about Claude Code or the Lindholm Code framework into the Obsidian knowledge vault.
disable-model-invocation: true
---

# Add To Vault

Captures the substantial explanations Claude gave in the current conversation — how Claude Code works, or how the Lindholm Code framework is built and used — into the Obsidian vault as well-formed notes.

The conversation is the source: not the user's own inputs, and not fresh-topic research — research only fills concrete gaps. Write only what is verified, and never break the vault's format. A note that states an unconfirmed claim as fact, or that reads unlike its neighbours, is a defect, not a contribution.

The vault holds two sections under one root — a general **Claude Code** section and a **Lindholm Code framework** section. One run writes to exactly one section; the skill asks which before building the candidate list.

## Step 1 — Scan the session for candidates

Read back over the current conversation for **substantial explanations** — passages where a concept, mechanism, or distinction was actually explained (how something works, why, what differs from what) — not task back-and-forth, quick factual answers, or the user's own statements. Judge this bar from the conversation alone; do not consult the vault to decide what qualifies.

A user statement is never a candidate on its own. Where one accurately describes a mechanism Claude also explained, fold it into that subject rather than making it a separate note.

If the session holds no substantial explanation, say so and stop — add nothing.

Done: a set of substantial-explanation subjects is identified, or the skill has stopped because there are none.

## Step 2 — Locate the vault

Resolve the vault root in order:

1. A path supplied in the invocation or command context.
2. The `$OBSIDIAN_CLAUDE_VAULT` environment variable.
3. Otherwise, ask the user for it, and suggest recording it in the command wrapper so it resolves automatically next time.

Confirm the resolved root contains `00 - Home.md` (the master index). If it does not, stop and report — this is not the vault.

Done: the vault root is resolved and validated by the presence of `00 - Home.md`.

## Step 3 — Choose the target section

The vault has two sections, and one run writes to exactly one of them. Read the root to find the two section folders, then ask the user which to write to as a numbered choice, closing with a single `Which? (1-2)` (the global interaction-design doctrine):

1. **Claude Code** — general, product-level knowledge of how Claude Code itself works: concepts, skills, hooks, agents, memory, rules, commands, workflow. Nothing project-specific and nothing framework-specific. Its folder is the general section (currently `01. Claude Code/`), holding the topic subfolders `Core Concepts`, `Skills`, `Hooks`, `Agents`, `Memory`, `Rules & Config`, `Commands`, `Workflow Optimization`; notes are tagged `#claude-code`.
2. **Lindholm Code framework** — how *this* framework is built and used: its agents, skills, rules, commands, lifecycle, and authoring conventions. Its folder is the framework section (currently `02. Lindholm Code Framework/`), with lifecycle skills under its `Meta-Skills/` subfolder; notes are tagged `#lindholm-code`.

Match the sections by role, not by exact name — take the general Claude Code folder and the framework folder as they actually appear at the root. Wait for the pick.

Then keep only the Step 1 candidates that belong to the chosen section. If a strong candidate clearly belongs to the *other* section, list it under a short "belongs to the other section" note and suggest re-running the skill with that section — do not capture it here. If nothing remains for the chosen section, say so (pointing to any other-section candidates) and stop.

Done: the target section is chosen, candidates are filtered to it, and any cross-section candidates are noted for a separate run.

## Step 4 — Build and present the candidate list

For each remaining candidate, do a **live read of the chosen section's files** (not a maintained register) to resolve two things:

- **Destination** — the specific folder or note within the chosen section. In the Claude Code section, place the note in the matching topic subfolder (`Core Concepts`, `Skills`, `Hooks`, `Agents`, `Memory`, `Rules & Config`, `Commands`, `Workflow Optimization`). In the framework section, place it in the framework folder (lifecycle skills in its `Meta-Skills/` subfolder). When a subject spans two folders within the section — a command and the mechanism behind it, say — its home is the note that *explains* it; the other folder gets a `[[wikilink]]`, not a copy.
- **New or update** — search the destination for a note already covering the subject. If one exists it is an update; otherwise a new note.

Present a numbered list, one line per candidate:

```
1. Permission model — auto-approve vs per-tool prompts · Core Concepts/ · UPDATE
2. add-to-vault design — scan/list/pick flow · Meta-Skills/ · NEW
```

Nothing is selected by default. Let the user pick one, several, or all — "all" being an explicit choice, never the default. Wait for the selection; write nothing before the user picks.

Done: each candidate has a resolved destination and new-vs-update decision, the list is shown, and the user has selected the subjects to capture.

## Step 5 — Verify before writing

For each selected subject, confirm each claim before it enters the vault. Annotate every fact with its true origin in the vault's style — `[from provided context]` or `[from training data]` for what was discussed, `[inference from ...]` for reasoning. If a concrete detail is missing, fill it silently by verifying against the tool's actual behaviour, official Anthropic docs, or the live configuration (framework claims against the store itself — `Rules/`, `Skills/`, `Agents/`, `Commands/`), annotated `[from Anthropic docs, verified <date>]`. A claim that cannot be confirmed is omitted or written with the canonical uncertain marker `[unverified — <what and why>]`; it is never stated as bare fact.

Done: every claim in every selected subject is sourced or flagged uncertain.

## Step 6 — Write to the vault's conventions

Match the format of the notes already in the vault exactly:

- A new note opens with a single `# Title` H1 (matching the file name), then one sentence stating what it covers.
- Sections are separated by `---` horizontal rules and headed with `##`.
- Cross-reference other notes with `[[wikilinks]]`, never paths or backticks.
- Facts carry the source annotation from Step 5.
- The note ends with a `## Related` section of `[[wikilinks]]`, then a final line of tags led by `#claude-code` (Claude Code section) or `#lindholm-code` (framework section), matching the full tag line used by existing notes in the same folder.

For a new note, write the whole note. For an update, merge in only what the existing note does not already say, in its existing voice; do not duplicate a fact already present (single source of truth).

Done: each selected subject is written or merged and reads like the notes already in the vault.

## Step 7 — Wire new notes into the index

A note nothing links to is an orphan. For a new note, index it the way sibling notes in the same section are reached — directly from `00 - Home.md` under its section (add a Quick Reference row if it fits that table), or, where that folder is indexed indirectly, through the hub or roster note that lists it (for example `Framework Skills` for a framework skill). Then add reciprocal `[[wikilinks]]` from the `## Related` sections of the closest existing notes. For an update to an already-indexed note, no new index line is needed — confirm the existing links still resolve.

Done: every written note is reachable from `00 - Home.md`, at least one sibling note back-links to each new note, and no orphan remains.

## Step 8 — Report

State in a few lines, per subject, what was added or updated, the file path, and any claim left flagged uncertain. Note any cross-section candidates deferred in Step 3 and the section to re-run for them.

Done: every change and its path are reported, with uncertainties surfaced.

## Relationship to other surfaces

- This vault is durable, human-facing documentation — distinct from Claude's file-based memory (`~/.claude/.../memory/`), which stores machine-recalled facts. Do not mirror content between them.
- It is also distinct from `graphify`, which builds a queryable knowledge graph from inputs. This skill writes hand-authored Obsidian notes; it does not generate a graph.
