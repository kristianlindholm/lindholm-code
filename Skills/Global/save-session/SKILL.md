---
name: save-session
description: Capture incomplete mid-session state to a per-project handoff file so a future session can resume exactly where this one stopped.
disable-model-invocation: true
---

# Save Session

Writes one handoff file under the project's `.claude/sessions/` capturing what this
session built, what worked, what failed, and the exact next step — so a future
session resumes with full context.

This is the mid-session, work-incomplete checkpoint. It writes only the session
file: it never edits `docs/PROGRESS.md`, never commits, never branches. Milestone
completion belongs to `wrap-it-up`, not here.

## Step 1 — Gather the session state

Collect, from this session:
- Files created or modified, and the current state of each (complete, in progress, broken, not started).
- What was attempted, what was decided, and why.
- Errors encountered and whether they were resolved.
- Current test/build status, if relevant.

Done when you can state, for every claim, whether it is confirmed (with evidence) or unverified.

## Step 2 — Resolve the destination

Ensure `.claude/sessions/` exists in the project root (create it if absent).

Choose the filename `YYYY-MM-DD-<slug>-session.md`:
- `YYYY-MM-DD`: today's date.
- `<slug>`: a 2-4 word lowercase-hyphen summary of the session's topic (for example `auth-cookie`, `save-session-design`). If the topic is genuinely unclear, use `session`.

Done when you have an absolute path that does not collide with an existing file.

## Step 3 — Write the file

Populate every section of the format below. Write each section honestly; if a section
has no content, write "None" — an incomplete file is worse than an honest empty section.

Every item under "What Worked" must carry its evidence (test passed, ran in browser,
endpoint returned 200). Without evidence, it belongs under "Not Tried Yet," not "Worked."

Done when the file exists with all sections present.

## Step 4 — Report, and confirm only if ambiguous

By default, write the file and report its path with a one-line summary. Do not ask for
confirmation on the normal path.

Pause and ask the user only when the captured state is genuinely ambiguous:
- You cannot tell whether an approach worked because there is no evidence.
- Signals conflict (a test passes but the feature visibly fails).
- The exact next step is unclear.

In those cases, show the affected sections and ask for the correction before finalizing.

Done when the file is written and either reported or corrected.

## Session file format

```markdown
# Session: YYYY-MM-DD — <topic>

**Project:** <name or path>
**Topic:** <one line>

## What We Are Building
<1-3 paragraphs; enough context for someone with zero memory of this session>

## What Worked (with evidence)
- <thing> — confirmed by: <specific evidence>
(or: "Nothing confirmed working yet.")

## What Did NOT Work (and why)
- <approach> — failed because: <exact reason / error message>
(or: "No failed approaches yet.")

## Not Tried Yet
- <promising approach not yet attempted>
(or: "None identified.")

## Current State of Files
| File | Status | Notes |
|------|--------|-------|
| `path` | Complete / In progress / Broken / Not started | <what it does / what's left> |
(or: "No files modified this session.")

## Decisions Made
- <decision> — reason: <why over the alternatives>
(or: "None.")

## Blockers & Open Questions
- <unresolved item>
(or: "None.")

## Exact Next Step
<the single most important thing to do on resume — precise enough to require zero re-thinking. Name the next action, not a tally of remaining work: never embed a count of open items; for the open granular set, point at docs/CHECKLIST.md>

## Environment Notes
<only if non-standard setup is needed to run the project; omit otherwise>
```

## Example

```markdown
# Session: 2026-06-29 — auth-cookie

**Project:** my-app
**Topic:** JWT auth via httpOnly cookie

## What Worked (with evidence)
- `/api/auth/register` — confirmed by: Postman POST returns 200, row visible in DB

## What Did NOT Work (and why)
- JWT in localStorage — failed because: SSR runs before localStorage exists, caused a hydration mismatch on every load

## Exact Next Step
In `app/api/auth/login/route.ts`, after generating the JWT, set it via
`cookies().set('token', jwt, { httpOnly: true })`, then verify the response carries a `Set-Cookie` header.
```

In a real file, write every section — the example omits the rest only for brevity.

## Failure modes

**Optimistic "Worked" claims.** Listing something under "What Worked" without evidence
poisons the next session, which trusts it. If you did not observe it pass, it goes under
"Not Tried Yet."

**Skipped sections.** Omitting a section reads as "nothing to report" when the truth may
be "unknown." Write "None" explicitly instead.

**Snapshotting an open-items count.** Writing "N open items" (or listing them) into the next
step duplicates `docs/CHECKLIST.md`, the single countable surface, and goes stale the moment
one closes — this handoff is never updated after the fact. Name the next action; point at the
checklist for the open set.

## Lifecycle

- `continue-project` detects the most recent `.claude/sessions/` file at session start and suggests resuming it.
- `wrap-it-up` deletes the session file when the work completes; if more than one exists, it lists them and asks rather than deleting by default.
