# Code Review Standards

## Purpose

Code review ensures quality, security, and maintainability before code is merged. This rule defines when and how to conduct code reviews.

## Code Review Rule

After completing a milestone implementation: always invoke the `code-reviewer` agent without asking. Pass the diff and the milestone's done-criteria as context.

After minor changes (small fixes, config edits, single-function changes): ask the user with a recommendation — "This change is small enough to skip review, but I recommend running one because [reason]. Review now? (Y/N)"

## When to Review

**MANDATORY review triggers:**

- After writing or modifying code
- Before any commit to shared branches
- When security-sensitive code is changed (auth, payments, user data)
- When architectural changes are made
- Before merging pull requests

**Pre-Review Requirements:**

Before requesting review, ensure:

- All automated checks (CI/CD) are passing
- Merge conflicts are resolved
- Branch is up to date with target branch

## Review Checklist

Before marking code complete:

- [ ] Code is readable and well-named
- [ ] Functions are focused (<50 lines)
- [ ] Files are cohesive (<800 lines)
- [ ] No deep nesting (>4 levels)
- [ ] Errors are handled explicitly
- [ ] No hardcoded secrets or credentials
- [ ] No console.log or debug statements
- [ ] Tests exist for new functionality
- [ ] Test coverage meets 80% minimum

## Security Review Triggers

**STOP and use security-reviewer agent when:**

- Authentication or authorization code
- User input handling
- Database queries
- File system operations
- External API calls
- Cryptographic operations
- Payment or financial code

## Review Severity Levels

| Level | Meaning | Action |
|-------|---------|--------|
| CRITICAL | Security vulnerability or data loss risk | **BLOCK** - Must fix before merge |
| HIGH | Bug or significant quality issue | **WARN** - Should fix before merge |
| MEDIUM | Maintainability concern | **INFO** - Consider fixing |
| LOW | Style or minor suggestion | **NOTE** - Optional |

## Agent Usage

Use these agents for code review:

| Agent | Purpose |
|-------|---------|
| **code-reviewer** | General code quality, patterns, best practices |
| **security-reviewer** | Security vulnerabilities, OWASP Top 10 |
| **typescript-reviewer** | TypeScript/JavaScript specific issues |
| **python-reviewer** | Python specific issues |
| **go-reviewer** | Go specific issues |
| **rust-reviewer** | Rust specific issues |

## Review Workflow

```
1. Run git diff to understand changes
2. Check security checklist first
3. Review code quality checklist
4. Run relevant tests
5. Verify coverage >= 80%
6. Use appropriate agent for detailed review
```

## Common Issues to Catch

### Security

- Hardcoded credentials (API keys, passwords, tokens)
- SQL injection (string concatenation in queries)
- XSS vulnerabilities (unescaped user input)
- Path traversal (unsanitized file paths)
- CSRF protection missing
- Authentication bypasses

### Code Quality

- Large functions (>50 lines) - split into smaller
- Large files (>800 lines) - extract modules
- Deep nesting (>4 levels) - use early returns
- Missing error handling - handle explicitly
- Mutation patterns - prefer immutable operations
- Missing tests - add test coverage

### Performance

- N+1 queries - use JOINs or batching
- Missing pagination - add LIMIT to queries
- Unbounded queries - add constraints
- Missing caching - cache expensive operations

## Approval Criteria

- **Approve**: No CRITICAL or HIGH issues
- **Warning**: Only HIGH issues (merge with caution)
- **Block**: CRITICAL issues found

## Receiving Review Feedback

Review feedback — whether from the user, from a review agent (`code-reviewer`, `security-reviewer`, a language reviewer), or from an external reviewer on a pull request — is a set of findings to evaluate technically, not orders to implement on sight. Verify before implementing, and acknowledge with the fix rather than with performative agreement. This is the sparring-partner stance applied to review: technical correctness over social comfort.

### Response pattern

For each finding:

1. Read the whole finding before reacting.
2. Restate the requirement in your own words to confirm you understood it.
3. Verify it against the codebase — is it correct for this code, this stack, this version?
4. Evaluate whether it holds technically here, or whether there is a reason (legacy, compatibility, a prior architectural decision) the current code is the way it is.
5. Respond: implement the fix, or push back with technical reasoning. Not both, not neither.
6. Implement one finding at a time and re-run the covering tests for each.

### No performative agreement

State the fix or the disagreement; do not perform enthusiasm or gratitude. The change in the code shows the feedback was heard.

| Avoid | Use instead |
|-------|-------------|
| "You're absolutely right!" | "Confirmed — [the specific issue]. Fixed in [location]." |
| "Great point!" / "Thanks for catching that!" | State what changed, or just make the change. |
| "Let me implement that now" (before verifying) | Verify against the codebase first, then implement. |

### Clarify before implementing

If any finding is unclear, stop and ask about all unclear items before implementing any of them. Findings are often related, and partial understanding produces a wrong fix. Do not implement the items you understood and defer the rest.

### Verify external feedback harder

- **From the user:** trusted — implement after you understand it; still ask if the scope is unclear.
- **From a review agent or external reviewer:** be skeptical and check carefully. Before implementing, confirm the suggestion is correct for this codebase, does not break existing behavior, and that the reviewer had full context. If it conflicts with a decision the user has already made, stop and raise it rather than silently overriding it.

### YAGNI-check suggested features

When a reviewer suggests "implementing this properly" — adding configuration, endpoints, export formats, or generality — check whether anything actually uses it. If nothing does, propose removing it rather than building it out: "Nothing calls this — remove it (YAGNI), or is there usage I am missing?"

### When to push back

Push back, with technical reasoning and reference to working code or tests, when a suggestion:

- breaks existing functionality,
- is technically incorrect for this stack or version,
- requires a feature nothing uses (YAGNI),
- exists for a legacy or compatibility reason, or
- conflicts with an architectural decision the user has made (raise it with the user).

If you pushed back and were wrong, state the correction factually and move on — no long apology, no defense of why you pushed back.

### Acting on findings

Resolve findings by severity (see Review Severity Levels): fix CRITICAL and HIGH before proceeding, fix MEDIUM when reasonable, note LOW. A confirmed bug finding gets a failing test first (the `tdd` skill) so the fix is proven and cannot regress.

## Integration with Other Rules

This rule works with:

- [testing.md](testing.md) - Test coverage requirements
- [security.md](security.md) - Security checklist
- [git-workflow.md](git-workflow.md) - Commit standards
- [agents.md](agents.md) - Agent delegation
