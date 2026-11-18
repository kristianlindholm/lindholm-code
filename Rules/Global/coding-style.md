# Coding Style

## Immutability (CRITICAL)

ALWAYS create new objects, NEVER mutate existing ones:

```
// Pseudocode
WRONG:  modify(original, field, value) → changes original in-place
CORRECT: update(original, field, value) → returns new copy with change
```

Rationale: Immutable data prevents hidden side effects, makes debugging easier, and enables safe concurrency.

## Core Principles

### KISS (Keep It Simple)

- Prefer the simplest solution that actually works
- Avoid premature optimization
- Optimize for clarity over cleverness

### DRY (Don't Repeat Yourself)

- Extract repeated logic into shared functions or utilities
- Avoid copy-paste implementation drift
- Introduce abstractions when repetition is real, not speculative

### YAGNI (You Aren't Gonna Need It)

- Do not build features or abstractions before they are needed
- Avoid speculative generality
- Start simple, then refactor when the pressure is real

## File Organization

MANY SMALL FILES > FEW LARGE FILES:
- High cohesion, low coupling
- 200-400 lines typical, 800 max
- Extract utilities from large modules
- Organize by feature/domain, not by type

## Authored governance documents

The limits above govern code, and the same ones apply here: 200-400 lines typical, 800 max. One
addition, because a document can be inside its size limit and still be unusable.

The cost of leaving documents unbounded has been measured once, and once is enough to cite: a
project `CLAUDE.md` reached 110,868 bytes from 3,835, across 80 commits of which 79 made it
larger and one made it smaller by 21 bytes.

**A file loaded into every session is an index, not a container.** `CLAUDE.md` holds what binds
*every* change, plus one line per topic pointing at the file that holds the rest. Detail lives in
`docs/rules/<practice>.md`, `docs/architecture/<subsystem>.md`, and `docs/adr/`, consulted when
the work touches them. This mirrors how `~/.claude/CLAUDE.md` indexes `~/.claude/rules/`:
**consult the relevant file rather than reproducing its content.**

**Bound the entry, not only the file.** A 20,000-byte rule is a problem wherever it is filed, and
splitting files does not touch it. A rule states what to do; the argument for it lives in the ADR
that decided it.

The bound is **40 lines per entry** — a trigger to look, like the 800-line file cap, not a hard
ceiling. Past it, check whether what has accumulated is the rule or the argument for the rule. If
it is the argument, it belongs in an ADR and the rule should point at it.

**Three things that are not rules and do not belong in one:**
- *Its own revision history* — "raised from 800, to 1400 on …, to 1500 on …". The rule states what
  is true now; the ADR carries why it changed.
- *A restatement of a surface that already owns it.* A rule that both points at an owning file and
  repeats its content has doubled the reading and created a second thing to keep true.
- *Narrative of the incident that prompted it.* Keep the instruction it produced.

**A standard nothing checks is a prose hint.** The runaway file above had a template asking for
"2-3 sentences"; that is why this standard names its enforcer. `wrap-it-up`'s truth pass reads
these documents back each wrap and reports what is over — it reports only, since changing a rule
goes through the same approval as writing one.

## Error Handling

ALWAYS handle errors comprehensively:
- Handle errors explicitly at every level
- Provide user-friendly error messages in UI-facing code
- Log detailed error context on the server side
- Never silently swallow errors

## Input Validation

ALWAYS validate at system boundaries:
- Validate all user input before processing
- Use schema-based validation where available
- Fail fast with clear error messages
- Never trust external data (API responses, user input, file content)

## Naming Conventions

- Variables and functions: `camelCase` with descriptive names
- Booleans: prefer `is`, `has`, `should`, or `can` prefixes
- Interfaces, types, and components: `PascalCase`
- Constants: `UPPER_SNAKE_CASE`
- Custom hooks: `camelCase` with a `use` prefix

## Code Smells to Avoid

### Deep Nesting

Prefer early returns over nested conditionals once the logic starts stacking.

### Magic Numbers

Use named constants for meaningful thresholds, delays, and limits.

### Long Functions

Split large functions into focused pieces with clear responsibilities.

## Code Quality Checklist

Before marking work complete:
- [ ] Code is readable and well-named
- [ ] Functions are small (<50 lines)
- [ ] Files are focused (<800 lines)
- [ ] No deep nesting (>4 levels)
- [ ] Proper error handling
- [ ] No hardcoded values (use constants or config)
- [ ] No mutation (immutable patterns used)

## Related rules

- [code-review.md](code-review.md) — the review checklist enforces these standards.
- [patterns.md](patterns.md) — design patterns that build on these principles.
