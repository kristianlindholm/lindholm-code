# Defense-in-Depth Validation

## Core principle

When a bug is caused by an invalid value, adding one check where you found it feels sufficient. But a single check is bypassed by a different code path, by later refactoring, or by a mock in a test. Validate at every layer the value passes through, so the bug becomes structurally impossible rather than merely fixed once.

## Why multiple layers

One validation says "we fixed the bug." Validation at each layer says "we made the bug impossible." Different layers catch different cases, and in practice each layer catches something the others miss:

- **Entry-point validation** rejects obviously invalid input at the boundary, catching most cases.
- **Business-logic validation** ensures the value makes sense for the specific operation, catching edge cases that pass the boundary.
- **Environment guards** refuse dangerous operations in contexts where they must never happen (for example, refusing a destructive filesystem operation outside a temp directory while tests run).
- **Diagnostic instrumentation** captures context before the dangerous operation, so when a future case slips through every other layer, the forensic trail already exists.

## Applying the pattern

After tracing a bug to its source (see [root-cause-tracing.md](root-cause-tracing.md)):

1. **Map the data flow.** List every point the value passes through, from where it originates to where it is finally used.
2. **Add a check at each layer.** Entry, business logic, environment guard, diagnostics — each rejecting or recording the invalid value in its own terms.
3. **Test that each layer holds independently.** Bypass the entry check and confirm the business-logic check still catches the value. A layer that is never exercised is not defense; it is dead code.

## Caution

Defense in depth is added _after_ the root cause is fixed, not instead of fixing it. Layers of validation around an unfixed cause hide the problem and make later debugging harder. Fix the source first; then make it impossible to reintroduce.

Keep the validations meaningful: each layer should reject a genuinely possible bad state, not restate the same check four times. Redundant identical checks add maintenance cost without adding coverage.
