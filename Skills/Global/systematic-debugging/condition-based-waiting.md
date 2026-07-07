# Condition-Based Waiting

## Core principle

Flaky tests often guess at timing with an arbitrary delay. This creates a race condition: the test passes on a fast machine and fails under load or in CI. Wait for the actual condition you care about, not for a guess about how long it takes to occur.

## When to use

- A test contains an arbitrary delay (a fixed sleep or timeout before asserting).
- A test is flaky — it passes sometimes and fails under load or in parallel.
- The code is waiting for an asynchronous operation to complete.

**Do not use when** the test is verifying timing behavior itself (a debounce or throttle interval). In that case the wait is the thing under test — keep it, and document why the specific duration is correct.

## The pattern

Replace "wait a fixed time, then check" with "poll the condition until it holds, up to a timeout":

```
BEFORE (guessing at timing):
  wait a fixed delay
  assert result is ready        # fails if the delay was too short

AFTER (waiting for the condition):
  wait until (result is ready), timeout T
  assert result is ready        # only proceeds once it is actually ready
```

A general waiter polls a condition function at a short interval and throws a descriptive error if a timeout elapses before the condition becomes true.

## Common targets

| Waiting for | Condition to poll |
|-------------|-------------------|
| An event | the event with the expected type has arrived |
| A state | the object or machine has reached the expected state |
| A count | the collection has reached the expected size |
| A file | the file exists at the expected path |
| A compound result | every required field is present and valid |

## Common mistakes

- **Polling too fast** wastes CPU. Poll at a small but real interval (on the order of ten milliseconds), not on every tick.
- **No timeout** means the test hangs forever if the condition is never met. Always include a timeout that throws a message naming what was awaited.
- **Caching the value before the loop** checks stale data. Read the value inside the loop so each poll sees the current state.

## When a fixed delay is genuinely correct

If a behavior is defined by a known interval (a tool that emits output every N milliseconds, and you need two emissions to verify partial output), first wait for the triggering condition, then wait the known duration with a comment explaining the arithmetic. The fixed wait is justified only when it follows a condition wait and is based on a documented, known timing — never as a guess.

## Concrete helpers

The general waiter and any reusable polling helpers belong in the relevant language testing skill (for example `react-testing`, `python-testing`, `e2e-testing`), where they can use the framework's own waiting primitives. This file describes the technique; the language skill supplies the implementation.
