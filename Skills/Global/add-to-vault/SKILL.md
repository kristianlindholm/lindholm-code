---
name: add-to-vault
description: Capture Claude's explanations from the current conversation about Claude Code or the Lindholm Code framework into the Obsidian knowledge vault.
disable-model-invocation: true
---

# Add To Vault

Captures the substantial explanations Claude gave in the current conversation — how Claude Code works, or how the Lindholm Code framework is built and used — into the Obsidian vault as well-formed notes.

The conversation is the source: not the user's own inputs, and not fresh-topic research — research only fills concrete gaps. Write only what is verified, and never break the vault's format. A note that states an unconfirmed claim as fact, or that reads unlike its neighbours, is a defect, not a contribution.

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

## Step 3 — Build and present the candidate list

For each candidate subject, do a **live read of the vault files** (not a maintained register) to resolve two things:

- **Destination** — general Claude Code mechanics go in the matching general folder (`Core Concepts`, `Skills`, `Hooks`, `Agents`, `Memory`, `Rules & Config`, `Commands`, `Workflow Optimization`) tagged `#claude-code`; framework specifics go in `Lindholm Code Framework/` (lifecycle skills in its `Meta-Skills/` subfolder) tagged `#lindholm-code`. When a subject spans two folders (a command and the mechanism behind it, say), its home is the note that *explains* it; the other folder gets a `[[wikilink]]`, not a copy.
- **New or update** — search the destination folder for a note already covering the subject. If one exists it is an update; otherwise a new note.

Present a numbered list, one line per candidate:

```
1. Permission model — auto-approve vs per-tool prompts · Core Concepts/ (general) · UPDATE
2. add-to-vault design — scan/list/pick flow · Lindholm Code Framework/Meta-Skills/ (framework) · NEW
```

Nothing is selected by default. Let the user pick one, several, or all — "all" being an explicit choice, never the default. Wait for the selection; write nothing before the user picks.

Done: each candidate has a resolved destination and new-vs-update decision, the list is shown, and the user has selected the subjects to capture.

## Step 4 — Verify before writing

For each selected subject, confirm each claim before it enters the vault. Annotate every fact with its true origin in the vault's style — `[from provided context]` or `[from training data]` for what was discussed, `[inference from ...]` for reasoning. If a concrete detail is missing, fill it silently by verifying against the tool's actual behaviour, official Anthropic docs, or the live configuration (framework claims against the store itself — `Rules/`, `Skills/`, `Agents/`, `Commands/`), annotated `[from Anthropic docs, verified <date>]`. A claim that cannot be confirmed is omitted or written with the canonical uncertain marker `[unverified — <what and why>]`; it is never stated as bare fact.

Done: every claim in every selected subject is sourced or flagged uncertain.

## Step 5 — Write to the vault's conventions

Match the format of the notes already in the vault exactly:

- A new note opens with a single `# Title` H1 (matching the file name), then one sentence stating what it covers.
- Sections are separated by `---` horizontal rules and headed with `##`.
- Cross-reference other notes with `[[wikilinks]]`, never paths or backticks.
- Facts carry the source annotation from Step 4.
- The note ends with a `## Related` section of `[[wikilinks]]`, then a final line of tags led by `#claude-code` (general) or `#lindholm-code` (framework), matching the full tag line used by existing notes in the same folder.

For a new note, write the whole note. For an update, merge in only what the existing note does not already say, in its existing voice; do not duplicate a fact already present (single source of truth).

Done: each selected subject is written or merged and reads like the notes already in the vault.

## Step 6 — Wire new notes into the index

A note nothing links to is an orphan. For a new note, index it the way sibling notes in the same folder are reached — directly from `00 - Home.md` under its section (add a Quick Reference row if it fits that table), or, where that folder is indexed indirectly, through the hub or roster note that lists it (for example `Framework Skills` for a framework skill). Then add reciprocal `[[wikilinks]]` from the `## Related` sections of the closest existing notes. For an update to an already-indexed note, no new index line is needed — confirm the existing links still resolve.

Done: every written note is reachable from `00 - Home.md`, at least one sibling note back-links to each new note, and no orphan remains.

## Step 7 — Report

State in a few lines, per subject, what was added or updated, the file path, and any claim left flagged uncertain.

Done: every change and its path are reported, with uncertainties surfaced.

## Relationship to other surfaces

- This vault is durable, human-facing documentation — distinct from Claude's file-based memory (`~/.claude/.../memory/`), which stores machine-recalled facts. Do not mirror content between them.
- It is also distinct from `graphify`, which builds a queryable knowledge graph from inputs. This skill writes hand-authored Obsidian notes; it does not generate a graph.
