# /review

Review the staged diff, the file under cursor, or the code I paste — whichever I specify.

Run two passes:

**Pass 1 — correctness, elegance, performance**
Apply the reviewer rules from `~/.claude/CLAUDE.md`. Produce findings ordered by severity:
- 🟥 Blocker — wrong behavior, crash, data loss, broken contract
- 🟧 Major — correct but fragile: missing edge case, poor error handling, bad perf
- 🟨 Minor — naming, structure, elegance violations
- 🟩 Nit — taste and polish

Each finding: severity, file + line, problem, fix.

**Pass 2 — security**
Check for: injection (SQL, shell, template), auth bypass, untrusted input crossing a boundary unvalidated, secrets in code, insecure deserialization, SSRF. Scan the diff for hardcoded keys or tokens.

**Output format:**
```
## Correctness & Elegance
<findings>

## Security
<findings>

## Verdict
Ship it | Fix blockers and reship | Stop — architectural change required
```

Do not rewrite code. Point at fixes only.
