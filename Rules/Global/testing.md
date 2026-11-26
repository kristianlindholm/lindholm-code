# Testing Requirements

## Minimum Test Coverage: 80%

Test Types (ALL required):
1. **Unit Tests** - Individual functions, utilities, components
2. **Integration Tests** - API endpoints, database operations
3. **E2E Tests** - Critical user flows (framework chosen per language)

## Test-Driven Development

MANDATORY workflow:
1. Write test first (RED)
2. Run test - it should FAIL
3. Write minimal implementation (GREEN)
4. Run test - it should PASS
5. Refactor (IMPROVE)
6. Verify coverage (80%+)

## Troubleshooting Test Failures

1. Use the `tdd` skill (test-first workflow)
2. Check test isolation
3. Verify mocks are correct
4. Fix implementation, not tests (unless tests are wrong)

## Skill Support

- **`tdd` skill** - Use PROACTIVELY for new features; enforces write-tests-first

## What a test costs to run

A test needing a browser, database, container, or server starts that resource **once per run**, not
once per test. Each test gets an isolated slice of the running resource — a fresh browser context, a
rolled-back transaction, a schema per worker. Reuse the process; never reuse the state.

Spawning per test is the obvious way to write a harness and is fine at ten tests. At two hundred the
startup dwarfs the assertions and the suite takes minutes. If a suite is already slow, use the
`speed-up-tests` skill, which measures before changing anything.

## Test Structure (AAA Pattern)

Prefer Arrange-Act-Assert structure for tests:

```typescript
test('calculates similarity correctly', () => {
  // Arrange
  const vector1 = [1, 0, 0]
  const vector2 = [0, 1, 0]

  // Act
  const similarity = calculateCosineSimilarity(vector1, vector2)

  // Assert
  expect(similarity).toBe(0)
})
```

### Test Naming

Use descriptive names that explain the behavior under test:

```typescript
test('returns empty array when no markets match query', () => {})
test('throws error when API key is missing', () => {})
test('falls back to substring search when Redis is unavailable', () => {})
```

## Related rules

- [code-review.md](code-review.md) — the 80% coverage gate in the review checklist.
- [development-workflow.md](development-workflow.md) — where the TDD step sits in the pipeline.
