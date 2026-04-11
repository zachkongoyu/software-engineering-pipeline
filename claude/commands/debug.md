# /debug

Reproduce a failure, name the root cause, write a regression test, and apply the minimal fix.

**Usage:** `/debug <failure message, stack trace, or description>`

Work through these steps in order. Do not skip or reorder them.

## 1 — Reproduce
Run the failing test or command. Capture the exact output. If you cannot reproduce it deterministically, that is your first problem — make it deterministic before anything else. Quote the failure verbatim.

## 2 — Bound
Narrow to the file, function, and line where the wrong thing happens. Read the code — do not skim. Show your reasoning: "the failure originates in X because Y."

## 3 — Hypothesize
One sentence: "I think X is wrong because Y." Commit to it.

## 4 — Check
Read the code, trace the data, run a minimal experiment. Do not apply the fix yet. If the check contradicts the hypothesis, form a new one. Never keep a hypothesis the evidence doesn't support.

## 5 — Name the root cause
One sentence — the cause, not the symptom. "The handler returns 200 when the repo raises `NotFound`" is a root cause. "It returns the wrong status" is a symptom.

## 6 — Regression test first
Write a test that fails on the current code for exactly the reason you named. Show it failing. This test is non-negotiable — it's what prevents the bug coming back.

## 7 — Minimal fix
Change only what the root cause requires. Show the regression test passing and the full suite green.

## 8 — Blast radius
Name any other callsites that share the same bug. Don't fix them here — note them.

**Output:**
```
Failure:      <verbatim output>
Root cause:   <one sentence>
Regression:   <test name> — FAIL before, PASS after
Suite:        <command>: N passed, 0 failed
Blast radius: <none | list>
```
