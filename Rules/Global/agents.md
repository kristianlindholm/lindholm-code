# Agent Orchestration

## Available Agents

Located in `~/.claude/agents/`:

| Agent | Purpose | When to Use |
|-------|---------|-------------|
| planner | Implementation planning | Complex features, refactoring |
| architect | System design | Architectural decisions |
| code-reviewer | Code review | After writing or modifying code |
| code-simplifier | Simplify and refine working code | After code passes, before review |
| security-reviewer | Security analysis | Before commits |
| typescript-build-resolver | Fix build and type errors | When the build fails |
| language-specific reviewers and build-resolvers | Per-language review and build fixes | e.g. `rust-reviewer`, `python-reviewer`, `react-reviewer` for that language |

For test-driven development, use the `tdd` skill (test-first workflow); there is no separate TDD agent.

## Immediate Agent Usage

No user prompt needed:
1. Complex feature requests - Use **planner** agent
2. Code just written/modified - Use **code-reviewer** agent
3. Bug fix or new feature - Use the `tdd` skill (test-first)
4. Architectural decision - Use **architect** agent

## Parallel Task Execution

Dispatch independent work concurrently — multiple agent calls in a single message run in parallel; one call per message runs sequentially.

```markdown
# GOOD: Parallel execution
Launch 3 agents in parallel:
1. Agent 1: Security analysis of auth module
2. Agent 2: Performance review of cache system
3. Agent 3: Type checking of utilities

# BAD: Sequential when unnecessary
First agent 1, then agent 2, then agent 3
```

Give each agent a focused scope, the context it needs to work alone, and the specific output you expect back (hand over and return bulk via files — see "Working with subagents efficiently").

Do not parallelize when the tasks share state (agents editing the same files conflict), when the failures may be related (one fix may resolve several — investigate together first), or when the work needs full-system context to understand.

When parallel agents return, check that their changes do not conflict, integrate them, and run the full test suite — independent agents can each make a correct change that clashes with another.

## Multi-Perspective Analysis

For complex problems, use split role sub-agents:
- Factual reviewer
- Senior engineer
- Security expert
- Consistency reviewer
- Redundancy checker

## Working with subagents efficiently

Dispatching an agent and reading its reply both spend the controller's context. Two practices keep a long run lean and recoverable.

**Hand over files, not pasted text.** When an agent needs a large artifact (a diff, a spec, a task brief), give it the path and let it read the file rather than pasting the content into the dispatch prompt — pasted text stays resident in the controller's context for the rest of the session. Likewise, have an agent that produces a large result (a plan, a review report) write it to a file and return only a short summary plus that path. The reviewer and build-resolver agents already work against the working tree and diff; this governs anything bulk you would otherwise inline.

Any handoff file written for this purpose is transient scratch, not a deliverable: write it under a gitignored scratch path (for example `.claude/scratch/`), and never commit it.

**Track long work durably.** Conversation memory does not survive context compaction. For a multi-step run, record completed, committed work where it persists — `PROGRESS.md` (`RESUME HERE`), the project checklist, and git history — not only in the running conversation. After a compaction or resume, trust those records and `git log` over recollection: never redo or re-dispatch work they already show as complete.

## Related rules

- [development-workflow.md](development-workflow.md) — where each agent slots into the pipeline.
- [code-review.md](code-review.md) — review delegation and the reviewer agents.
- [performance.md](performance.md) — model selection per agent role.
