---
name: debug
description: Reproduce a failure, isolate the root cause, write a regression test, and apply the minimal fix. Never guesses — forms a hypothesis, checks it, revises until the cause is named.
agent: agent
argument-hint: "Failure message or stack trace"
tools: [search, read, edit, execute, web]
---

Debug the following failure following `copilot/agents/debugger.agent.md`.

**Failure:** $FAILURE_OR_STACK_TRACE

Work through these steps in order. Do not skip any.

1. **Reproduce** — run the failing test or command and capture the exact
   output. If you cannot reproduce it, stop and say so.

2. **Bound** — narrow to the file, function, and line where the wrong
   thing happens. Show your reasoning.

3. **Hypothesize** — one sentence: "I think X is wrong because Y."

4. **Check** — read the code, trace the data, run a targeted experiment.
   Never apply the fix yet.

5. **Name the root cause** — one sentence that describes the cause, not
   the symptom.

6. **Regression test first** — write a test that fails on the current
   code for exactly the reason you named. Show it failing.

7. **Minimal fix** — change only what the root cause requires. Show the
   regression test passing and the full suite green.

8. **Blast radius** — name any other callsites that share the same bug.
   Don't fix them; note them.

## Output format

```
## Failure
<exact output>

## Root cause
<one sentence>

## Regression test
<test name> — FAIL before fix, PASS after

## Fix
<unified diff>

## Suite
<command>: N passed, 0 failed

## Blast radius
<"none" or list of locations>
```
