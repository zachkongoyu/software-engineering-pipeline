# /tdd

Drive the change I describe test-first: Red → Green → Refactor.

**Usage:** `/tdd <behavior to implement>`

## Step 1 — Red
Write test(s) describing the behavior. Follow the test rules in `~/.claude/CLAUDE.md`:
- Name describes behavior, not the function.
- One behavior per test. Fakes over mocks. No sleep, no network, no wall clock.

Run the suite. Show the new tests **failing for the right reason** (assertion failure, not import error). Quote the output verbatim.

## Step 2 — Green
Write the **minimal** implementation that makes the tests pass. Nothing more:
- Smallest change that satisfies the test.
- Elegance bar applies — but do not over-engineer to pass one test.

Run the full suite. Show all green. Quote verbatim.

## Step 3 — Refactor
With behavior locked in, clean up:
- Rename anything unclear.
- Extract any duplication (rule of three).
- Improve types.

Run suite again — must stay green.

**Output:**
```
Red:      <test names> — FAIL: <reason>
Green:    <command> — PASS: N tests
Refactor: <what changed> — PASS: N tests

Changed:
  - <file>: <summary>
```
