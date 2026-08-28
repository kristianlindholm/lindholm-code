# Development Workflow

> This file extends [git-workflow.md](git-workflow.md) with the full feature development process that happens before git operations.

The Feature Implementation Workflow describes the development pipeline: research, planning, TDD, code review, and then committing to git.

## Feature Implementation Workflow

0. **Research & Reuse** _(mandatory before any new implementation)_
   - **GitHub code search first:** Run `gh search repos` and `gh search code` to find existing implementations, templates, and patterns before writing anything new.
   - **Library docs second:** Use Context7 or primary vendor docs to confirm API behavior, package usage, and version-specific details before implementing.
   - **Exa only when the first two are insufficient:** Use Exa for broader web research or discovery after GitHub search and primary docs.
   - **Check package registries:** Search npm, PyPI, crates.io, and other registries before writing utility code. Prefer battle-tested libraries over hand-rolled solutions.
   - **Search for adaptable implementations:** Look for open-source projects that solve 80%+ of the problem and can be forked, ported, or wrapped.
   - Prefer adopting or porting a proven approach over writing net-new code when it meets the requirement.

1. **Plan First**
   - Use **planner** agent to create implementation plan
   - Generate planning docs before coding: PRD, architecture, system_design, tech_doc, task_list
   - Identify dependencies and risks
   - Break down into phases

2. **TDD Approach**
   - Use the `tdd` skill (test-first workflow)
   - Write tests first (RED)
   - Implement to pass tests (GREEN)
   - Refactor (IMPROVE)
   - Verify 80%+ coverage
   - When a test fails unexpectedly, a build breaks, or behavior is wrong, use the `systematic-debugging` skill before attempting any fix — find the root cause first, do not patch the symptom

3. **Code Review**
   - Use **code-reviewer** agent immediately after writing code
   - Address CRITICAL and HIGH issues
   - Fix MEDIUM issues when possible

4. **Commit & Push**
   - Detailed commit messages
   - Follow conventional commits format
   - See [git-workflow.md](git-workflow.md) for commit message format and PR process

5. **Pre-Review Checks**
   - Verify all automated checks (CI/CD) are passing
   - Resolve any merge conflicts
   - Ensure branch is up to date with target branch
   - Only request review after these checks pass

## Verification Before Completion

Evidence comes before any claim of success. Do not state that work is done, a test passes, a build is green, or a bug is fixed until you have run the command in the current state and read its output. The implementation skills' phrase "confirmed with evidence, not assumed" is this standard applied at their gates.

Before any completion or success claim:

1. Identify the command that would prove the claim.
2. Run it fresh and in full.
3. Read the output — exit code, failure count, the actual result — not a prior run.
4. State the claim only if the output confirms it; otherwise report the actual state.

This applies to every form of the claim — "done", "fixed", "passing", "ready" — and to any expression of satisfaction that implies them.

| Excuse | Reality |
|--------|---------|
| "It should work now." | Run the command; "should" is not evidence. |
| "I'm confident it passes." | Confidence is not output. Run it. |
| "The linter passed." | The linter is not the compiler or the tests. |
| "The agent reported success." | Verify independently — check the diff and run the command. |
| "A partial check is enough." | Partial proves only the part you ran. |
| "I already ran it earlier." | Code changed since. A stale run does not count. |

Red flags that you are about to claim without evidence: using "should", "probably", or "seems to"; expressing satisfaction ("done", "perfect") before running anything; being about to commit or open a PR without a fresh run; trusting an agent's success report over the diff and a re-run.

## Changing files

Use the Write and Edit tools for any change to a file that is part of the work, in preference to a shell command. They match the file's existing line endings, and they **fail loudly** when a search string does not match.

A scripted edit does not. `sed`, a PowerShell `-replace`, or a Python find-and-replace that matches nothing changes nothing and **exits successfully** — the command reports success and the edit never happened. That is the section above defeated at the tool level: the exit code is not evidence the file changed.

The usual reason a search silently fails to match is line endings. A text file ends each line with either LF (one byte, used by Linux, macOS and git's own storage) or CRLF (two bytes, the Windows convention). A search string built with one will not match a file written with the other. Measured on Windows, writing identical content through each path:

| Path | Endings produced |
|------|------------------|
| Write / Edit tools | LF (Edit matches the file it is editing) |
| bash `printf`, heredoc (`cat > f <<'EOF'`) | LF |
| python `open(f,'wb')`, or `open(f,'w',newline='')` | LF |
| python `open(f,'w')` — text mode | CRLF |
| PowerShell `Set-Content`, `Out-File`, `Add-Content` | CRLF |
| PowerShell `>` with a newline inside the string | **mixed** — CRLF and LF in one file |

**Mixed endings in a single file is the one genuinely damaging state.** No single ending is there for a later search to match, and git cannot report it: checkin normalises mixed, all-CRLF and all-LF to the same blob, so `git status` and `git diff` show the file clean forever. A file that is uniformly CRLF is not damage — an edit that matches its endings works — and it needs no repair.

When a scripted edit is genuinely the right instrument, such as a bulk change across many files:

- **Before:** read the file's dominant line ending and build both the search anchor and the replacement to match it. Never write a lone LF into a CRLF file.
- **After:** print the byte delta and the resulting ending counts. A zero delta where something should have changed is a silent no-match, not a no-op.
- Pair the byte delta with an explicit count of sites changed. A same-length replacement is +0 bytes and reads as a failure when it in fact worked.

Do not diagnose line endings with git — see `code-review.md` and the `code-reviewer` agent's false-positive list for why `git -c core.autocrlf=false diff` manufactures the problem it appears to reveal. Count the bytes instead.
