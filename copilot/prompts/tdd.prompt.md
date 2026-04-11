---
name: tdd
description: Drive a change test-first. Red → Green → Refactor. Takes a behavior description and produces a failing test, then the minimal implementation that makes it pass, then a cleanup pass.
agent: agent
argument-hint: "Behavior to build"
tools: [search, read, edit, execute]
---

Drive the following change test-first using the Red → Green → Refactor loop.

**What I want built:** $BEHAVIOR

## Step 1 — Red

Write the test(s) that describe `$BEHAVIOR`. Follow `copilot/instructions/tests.instructions.md`:
- Test names describe the behavior.
- One behavior per test.
- Fakes over mocks.
- Deterministic — no sleep, no network, no wall clock.

Run the suite. Show that the new tests **fail** for the right reason (not
"module not found", not a type error — a real assertion failure that says
the behavior doesn't exist yet). Quote the failure output verbatim.

## Step 2 — Green

Write the **minimal** implementation that makes the failing tests pass.
Follow `copilot/agents/implementer.agent.md`:
- Smallest change that satisfies the test.
- No gold-plating, no speculative generality.
- Elegance bar applies (`copilot-instructions.md`).

Run the full suite. Show that all tests pass. Quote the result verbatim.

## Step 3 — Refactor

Now that the behavior is locked in, clean up:
- Rename anything that isn't clear.
- Extract any duplication (rule of three applies).
- Verify the types say what the code does.
- Run the suite again. It must stay green.

Follow `copilot/agents/refactorer.agent.md`: behavior-preserving only.

## Output

```
Red:    <test name(s)> — FAIL: <failure reason>
Green:  <test command> — PASS: N tests
Refactor: <what changed> — PASS: N tests

Changed files:
  - <file>: <one-line summary>
```
