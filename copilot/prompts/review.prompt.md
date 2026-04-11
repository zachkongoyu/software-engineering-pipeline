---
name: review
description: Run the full reviewer + security pipeline on the current diff or selected code. Produces an ordered findings list and a verdict.
agent: agent
tools: [search, read, web]
---

Run a two-pass review on the code I have selected or the current staged diff.

**Pass 1 — correctness, elegance, performance** (reviewer agent)
Follow the rules in `copilot/agents/reviewer.agent.md`. Produce findings
ordered by severity (🟥 Blocker → 🟧 Major → 🟨 Minor → 🟩 Nit). Every
finding must have: severity, file + line, problem, fix.

**Pass 2 — security** (security agent)
Follow the rules in `copilot/agents/security.agent.md`. Check all ten
OWASP categories. Scan for secrets. Check any new dependencies.

Combine both passes into a single output:

```
## Review

### Correctness & Elegance
<findings from Pass 1, severity-ordered>

### Security
<findings from Pass 2>

### Supply chain
<deps added/changed + any CVEs>

### Secrets scan
<result>

## Verdict
<"Ship it" | "Fix blockers and reship" | "Stop — architectural change required">
```

Do not rewrite any code. Point at fixes; the implementer applies them.
