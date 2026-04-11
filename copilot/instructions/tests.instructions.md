---
applyTo: "**/{test,tests,spec,__tests__}/**,**/*.{test,spec}.{ts,tsx,js,jsx,mjs},**/*_test.go,**/test_*.py,**/*.test.rs"
description: Cross-language test rules. Auto-attached to every test file. Sits on top of the language-specific instructions and copilot-instructions.md.
---

# Tests — Cross-language Rules

These rules apply to every test file regardless of language. They sit on
top of the per-language instructions. When they conflict, these win for
test files specifically.

## The contract

Tests are the specification. They describe what the system does from the
outside. A test that breaks after an internal refactor that preserved
behavior was testing the wrong thing.

## Anatomy of a good test

```
arrange  — set up state, inputs, fakes
act      — call the one thing under test
assert   — check the one outcome
```

One behavior per test function. One `act` per test function. If you need
two `act` steps to test something, you're testing two things — split them.

## Naming (all languages)

The test name completes the sentence **"it [name]"**:

| Bad | Good |
|-----|------|
| `test_parse_1` | `parse_config_rejects_empty_host` |
| `TestHandleRequest` | `TestHandleRequest_returns404_when_user_not_found` |
| `it('works')` | `it('returns the cached value on second call')` |
| `test_validate` | `test_validate_raises_on_negative_quantity` |

## What to test (priority order)

1. **Error paths before happy paths.** Empty, null/nil/None, zero, one,
   negative, unicode, boundary values. The happy path rarely breaks in
   production; the edges always do.
2. **Contracts, not implementations.** Test what the function promises,
   not how it achieves it. If the test would break after renaming a
   private helper, it's testing the wrong layer.
3. **Failure modes at trust boundaries.** Every place untrusted data
   enters — HTTP handler, CLI arg, file parse, deserializer — must have
   tests for malformed, oversized, and adversarial input.
4. **Concurrency hazards.** If code shares mutable state across
   goroutines/threads/tasks, test with `-race` (Go), `--test-threads=1`
   (Rust), or `AsyncMock` + multiple callers (Python).
5. **Round-trips and invariants.** Serializers, parsers, encoders:
   `encode(decode(x)) == x`. Sorted collections: output is sorted.

## Fakes over mocks

Write a hand-rolled in-memory implementation of the interface under test.
A `FakeUserRepo` that stores users in a `dict` / `map` / `HashMap` is:
- Faster to read than a mock setup.
- Easier to debug when it fails.
- Reusable across tests.
- Honest about the contract — it either implements the interface or it
  doesn't compile.

Use a mock library only for collaborators you cannot fake cheaply (third-
party SDKs with 30 methods, OS interfaces).

## Determinism (non-negotiable)

- **No `time.sleep` / `Thread.sleep` / `tokio::time::sleep` in tests.**
  If you need time to pass, inject a fake clock.
- **No wall-clock timestamps.** Inject `datetime.now` / `time.Now` /
  `SystemTime::now` as a parameter or use a clock interface.
- **No randomness.** Seed explicitly or inject an RNG.
- **No real network.** Fake the HTTP client at the boundary.
- **No temp-file paths that include timestamps or PIDs.** Use
  `tmp_path` (pytest) / `t.TempDir()` (Go) / `tempfile::tempdir()` (Rust).

## Assertion style

- Assert on the **outcome**, not on the internal state: `assert response.status == 200`,
  not `assert handler.was_called`.
- Use **specific matchers**: `assert_matches_pattern`, `contains`,
  `raises(ConfigError)` — not a bare `assert True`.
- Include a **failure message** for non-obvious assertions so that the
  test output tells you what went wrong without having to re-read the code.

## Test file layout

```
tests/
  unit/         pure logic, no IO
  integration/  real DB/HTTP, real filesystem
  e2e/          full stack, runs against a deployed instance
```

Unit tests run in every `pre-commit` hook. Integration tests run in CI on
every push. E2E tests run in CI on every merge to the main branch.

## What a test must never do

- **Never disable the test to make CI green.** Quarantine it with a note
  and a tracker link.
- **Never assert on log output.** Logs are for humans; assertions are for
  contracts. If the behavior matters, expose it through a return value or
  an observable side effect.
- **Never share mutable state between test functions.** Each test starts
  clean; fixtures that need teardown use the framework's teardown hook.
- **Never write a test that passes vacuously** — e.g., an empty parametrize
  list, an `assert True`, or a `for` loop with zero iterations.
