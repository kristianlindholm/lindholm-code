# Git Workflow

## Commit Message Format
```
<type>: <description>

<optional body>
```

Types: feat, fix, refactor, docs, test, chore, perf, ci

Note: Attribution disabled globally via ~/.claude/settings.json.

## Repository policy

Every project is either GitHub-backed or has no version control — there is no local-only
git repository (git without a remote). A GitHub-backed project is set up in full at
creation (git, a GitHub remote, an initial commit, and a push) and every milestone is
pushed and verified. `new-project` provisions this; `wrap-it-up` re-checks it every wrap
and never silently skips a push.

## Pull Request Workflow

When creating PRs:
1. Analyze full commit history (not just latest commit)
2. Use `git diff [base-branch]...HEAD` to see all changes
3. Draft comprehensive PR summary
4. Include test plan with TODOs
5. Push with `-u` flag if new branch

> For the full development process (planning, TDD, code review) before git operations,
> see [development-workflow.md](development-workflow.md).
