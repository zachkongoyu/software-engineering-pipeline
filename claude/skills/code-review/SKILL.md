---
name: code-review
description: Code review skill. Critiques a diff or selected code for correctness, security, performance, and elegance. Produces findings ordered by severity with concrete fixes. Also checks for security issues — injection, untrusted input crossing boundaries, secrets, insecure deps. Invoke when reviewing staged changes, a PR, or any code before it ships.
---

# Code Review

A good review is honest, specific, and ordered by what actually matters. Its job is to stop real problems from shipping and to raise the craftsmanship bar — not to assert taste, not to demonstrate knowledge, not to inflate a finding count.

## Severity definitions

- **🟥 Blocker** — wrong behavior, crash, data loss, race condition, security hole, broken API contract, test that passes for the wrong reason. Cannot ship.
- **🟧 Major** — correct but fragile: missing edge case, poor error handling, O(n²) where O(n) is easy, unclear contract, dead code path, missing test for something that will regress.
- **🟨 Minor** — elegance and maintainability: naming, structure, premature abstraction, violations of the elegance principles that don't change behavior.
- **🟩 Nit** — taste and polish. Optional. Don't let nits crowd out real findings.

## Output format

```
## Review

### 🟥 Blocker — <title>
File: path/to/file.ext:LINE
Problem: <what's wrong>
Fix: <the smallest change that resolves it>

### 🟧 Major — ...
### 🟨 Minor — ...
### 🟩 Nit — ...

### Security
<findings, or "nothing flagged">

### Secrets scan
<result — quote any suspect line verbatim>

## Verdict
Ship it | Fix blockers and reship | Stop — architectural change required
```

## What to look for (ordered by what bites hardest)

### 1. Correctness — does it actually work?

Walk through the logic with real values. Then the unhappy paths: empty, nil/null/None, zero, one, many, concurrent, partial failure, timeout.

- Does every error path reach the caller?
- Can this panic / throw / crash on valid input?
- Is the output deterministic given the same input?
- Are boundary conditions fenced (off-by-one, inclusive/exclusive)?

### 2. Concurrency and ordering

- Shared mutable state without a lock?
- Lock acquisition order (deadlock potential)?
- Async without backpressure or cancellation?
- Goroutine / task with no defined lifetime?
- Iterator that mutates the collection it walks?

### 3. Security

Go through these every time, not just when the brief mentions security:

- **Injection** — untrusted input used in a SQL query, shell command, template, or path without sanitization.
- **Auth bypass** — can a caller reach a resource they shouldn't? Is authz checked before or after the expensive operation?
- **Untrusted input at a boundary** — HTTP handler, CLI arg, deserialized payload — is it validated and typed before use?
- **Secrets in code** — API keys, tokens, passwords, PEM blocks hardcoded or in a log call.
- **SSRF** — user-controlled input becoming a URL the server fetches.
- **Insecure defaults** — CORS `*`, debug endpoints, verbose error messages leaking stack traces.

### 4. Error handling

- Swallowed exceptions (`except: pass`, `if err != nil { _ = err }`)?
- Error logged and silently returned `nil` / `null` — the caller thinks it succeeded?
- Errors that lose the original cause (no wrapping / chaining)?
- Panics in library code?
- Generic error type where a specific one would let the caller recover?

### 5. Performance (only where it actually matters)

- N+1 queries (loop that issues a DB call per iteration)?
- Loading an unbounded result set into memory?
- O(n²) or worse in a hot path?
- Allocating inside a tight loop when a pool or pre-allocation would help?

Do not flag theoretical performance issues in cold paths. Only call it out when the scale makes it real.

### 6. Contracts and API surface

- Public signature changed without updating callers and docs?
- Config key added without a default or migration path?
- Error message the caller has to parse (string matching instead of typed errors)?
- Returned type that forces the caller to know about internal implementation?

### 7. Tests

- New behavior with no tests? Call it out as Major.
- Test that asserts on a private implementation detail? Call it out.
- Test that passes vacuously (empty loop, `assert True`, wrong mock setup)?
- Flaky test disguised as a real one (sleep, wall clock, network)?

### 8. Elegance (the principles from `~/.claude/CLAUDE.md`)

Every violation gets a Minor unless it's already caught as a higher severity:

- Functions doing more than one thing.
- Names that need a comment to be understood.
- Mutable shared state where immutable would work.
- Stringly-typed values where an enum would work.
- Defensive code against impossible states (trust the types).
- TODOs without a reason and a tracker link.

### 9. Scope

- Diff touches files unrelated to the stated change? Flag as Blocker — separate it.
- Refactor mixed with behavior change? Flag as Major — split the commits.

## Hard rules

1. **Never speculate.** If you can't trace the data flow to confirm a bug, say "unclear — trace needed" rather than guessing.
2. **Never rewrite the code in the review.** Point at the fix in one sentence. The implementer applies it.
3. **Never pad the review.** A short review with one real Blocker is worth ten reviews full of Nits. If there's nothing wrong, say "Ship it" and stop.
4. **Secrets scan is mandatory.** Always check the diff for patterns matching keys, tokens, passwords, connection strings. Report even if it looks fake — rotate it anyway.
5. **Verdict is one of three exact strings.** No hedging, no "mostly okay but...".
