---
name: testing-standards
description: >
  Testing quality standards for React/Next.js/React Native repositories.
  Enforces integration-style testing, behavior-driven structure, strong assertions,
  and stable selectors. Apply whenever writing or reviewing tests. Composes with /tdd.
---

# Testing Standards

## Philosophy

Tests verify **behavior from the user's POV**, not implementation internals.

A test should survive a full internal refactor. If renaming a hook breaks a test but the UI still works — the test was verifying implementation, not behavior.

See [mocking.md](mocking.md) for HTTP mocking rules, [assertions.md](assertions.md) for quality rules, [structure.md](structure.md) for describe hierarchy, [selectors.md](selectors.md) for DOM query priority.

---

## Integration Testing Mandate

Prefer integration-style tests: render the real component, mock only at HTTP boundaries, assert on rendered output.

```
User action → real hook → HTTP mock intercept → rendered output → assert
```

This catches: hook logic bugs, mapper bugs, loading/error state gaps, cache behavior.

Unit tests that mock hooks catch none of the above — and hide all of it.

---

## Workflow

### Before writing tests

1. Identify **user-facing behaviors**: happy path, error state, edge case
2. Pick the highest-level entry point (page → component → hook)
3. Set up HTTP mocks for every API the component calls

### Tracer bullet

Write ONE test that exercises the full stack end-to-end before writing the rest. If it passes, the wiring is correct. Then add variants.

### Coverage is a side-effect

Never start from "which lines are uncovered." Start from "which behaviors are untested." Coverage follows from good tests — not the other way around.

---

## Checklist Before Committing

- [ ] No mocks of your own hooks or services — see [mocking.md](mocking.md)
- [ ] No fake passes (`expect(true).toBe(true)`, silent conditionals) — see [assertions.md](assertions.md)
- [ ] No line numbers or branch names in test names — see [structure.md](structure.md)
- [ ] No CSS class or tag-name selectors — see [selectors.md](selectors.md)
- [ ] Every `beforeEach` resets mocks, cache, and shared state
- [ ] All async assertions use `findBy*` or `waitFor`
