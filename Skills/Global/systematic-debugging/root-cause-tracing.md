# Root-Cause Tracing

## Core principle

A bug usually surfaces deep in the call stack — a file written to the wrong path, a database opened with the wrong handle, an operation run in the wrong directory. The instinct is to fix it where the error appears. That is treating a symptom. Trace backward through the call chain until you find where the bad value originated, and fix it there.

## When to use

- The error happens deep in execution, not at an entry point.
- The stack trace shows a long call chain.
- It is unclear where the invalid value came from.
- You need to find which caller or test triggers the problem.

If you cannot trace backward (the chain dead-ends or crosses a process boundary you cannot inspect), fix at the closest point you can verify, and add instrumentation so the next occurrence reveals more.

## The tracing process

1. **Observe the symptom.** Note the exact failing operation and the bad value it received.
2. **Find the immediate cause.** Identify the line that directly performs the failing operation.
3. **Ask what called it.** Walk up the call chain one level at a time.
4. **Track the value, not just the calls.** At each level, ask what value was passed and whether it was already wrong here. A value such as an empty string often resolves silently to a dangerous default (the current working directory, the root path) instead of failing loudly.
5. **Find the original trigger.** Keep going until you reach the point where the bad value first entered the system. That is the root cause. Fix it there.

## Adding instrumentation when manual tracing stalls

When the chain is not obvious from reading, log just before the dangerous operation rather than after it fails. Capture the suspect value, the surrounding context (working directory, relevant environment variables, a timestamp), and a captured stack trace so the full call chain is visible. Run once, read the output, and identify the level where the value first goes wrong.

In test runs, write this diagnostic output to a stream that the test runner does not suppress — many runners hide the normal log channel, so a standard error or console channel is more reliable.

## Finding which test causes shared-state pollution

When something appears during a test suite but you cannot tell which test created it, bisect: run tests in halves (or one at a time) until the first run that produces the pollution is isolated. The culprit is the smallest set that still reproduces it. Once isolated, apply the tracing process to that single test.

## Pair with defense in depth

Fixing at the source removes the cause. Adding validation at each layer the value passed through (see [defense-in-depth.md](defense-in-depth.md)) makes the same bug impossible to reintroduce later through a different code path.
