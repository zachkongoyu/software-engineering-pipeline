---
name: tdd
description: Test-Driven Development skill. Drives changes using the Red → Green → Refactor loop. Write the failing test first, then the minimal code to pass it, then clean up. Never writes production code before a failing test exists. Invoke when implementing a feature, fixing a bug, or adding behavior to existing code.
---

# TDD — Test-Driven Development

TDD is a discipline, not a style preference. The loop is short and strict:

```
Red   → write a test that fails for the right reason
Green → write the minimal code that makes it pass
Refactor → improve the shape without changing the behavior
```

The point is not the tests themselves — it's that writing the test first forces you to think about the interface before the implementation, which almost always produces a better design.

## The loop in detail

### Red — write the test first

Before touching any production code:

1. Name the test after the behavior, not the function.
   `test_parse_rejects_negative_port`, not `test_parse_1`.
2. Write the test as if the ideal API already exists — the one you wish you had. If the test is awkward to write, the API is wrong.
3. Run it. It must fail. Check that it fails *for the right reason*: a missing function, a wrong return value, an uncaught error — not a syntax error or an import failure.
4. Quote the failure output verbatim. "It failed" is not enough.

**If the test passes immediately** without any production code, you wrote the wrong test. Stop and rethink what behavior you're pinning down.

### Green — write the minimum

Make the test pass with the least code possible:

- No extra parameters.
- No extra abstractions.
- No "I might need this later" logic.
- If hardcoding the return value makes the test pass, hardcode it — the next test will force you to generalize.

Run the full suite. Every existing test must still be green. Quote the result.

**If you need to change a test to make it pass**, stop. Either the new code broke a real behavior (fix the code, not the test), or the old test was pinned to an implementation detail (rewrite the test to assert on behavior, then proceed).

### Refactor — improve the shape

The test is green. The behavior is locked. Now make the code clear:

- Rename anything that doesn't read as prose.
- Extract a helper if the same three lines appeared before (rule of three).
- Remove the hardcoded value and replace with the real logic — if the next test forces it, let it force it.
- Tighten the types.

Run the suite. It must stay green. If it goes red, you changed behavior — revert the refactor and try a smaller step.

**Never refactor in the Green step and never add behavior in the Refactor step.**

## Test quality bar (enforced on every test you write)

- **Arrange / Act / Assert** — visible at a glance, in that order.
- **One behavior per test** — if you write "and" in the test name, split it.
- **Fakes over mocks** — a hand-written `FakeUserRepo` is clearer and more honest than any mock setup.
- **Deterministic** — no `time.now()`, no network, no randomness the test doesn't control.
- **Fast** — unit tests run in milliseconds. If a test is slow, it belongs in integration.
- **Fails for the right reason** — the assertion message alone tells you what broke.

## When to use which test level

| Level | What it covers | Speed | When |
|---|---|---|---|
| Unit | Pure logic, one function or class | < 1ms | Always, for the core |
| Integration | Real DB, real HTTP, real filesystem | < 500ms | At the boundary |
| E2E | Full stack, deployed instance | Seconds | On merge to main |

TDD lives at the unit level for the core and the integration level for boundaries. Do not TDD your E2E tests — they're too slow for the loop.

## Property-based testing (reach for it early)

When the domain has a property — a parser that round-trips, a sort that's stable, a codec that's invertible — use property-based tests instead of (or alongside) example-based tests. A few dozen random inputs will find edge cases no hand-written example would.

- **Python** — `hypothesis`
- **TypeScript** — `fast-check`
- **Go** — `testing/quick` or `gopter`
- **Rust** — `proptest`

## Bug fix protocol (TDD for bugs)

A bug fix that doesn't start with a failing test is a bug waiting to come back.

1. Write a test that reproduces the bug and fails on the current code.
2. Confirm it fails for the right reason (the bug, not something else).
3. Fix the code — minimal change, scoped to the root cause.
4. Confirm the test now passes and the suite is green.
5. That test is now a permanent regression guard — never delete it.

## Red flags (stop and ask)

- You've written more than ~20 lines of production code without a failing test. Stop. Step back. Write the test for what you just did.
- A test is hard to write. This means the design is wrong. The test is telling you something — listen to it before forcing it to pass.
- You're mocking five things to test one thing. You've coupled too much. The real fix is to split the code, not to mock harder.
- You feel the urge to write `@pytest.mark.skip` or `t.Skip()`. Don't. Either the test is wrong and needs to be rewritten, or the code is broken and needs to be fixed.
