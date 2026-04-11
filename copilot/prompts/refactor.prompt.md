---
name: refactor
description: Improve selected code without changing its behavior. Before/after story, behavior-preserving diff, suite green before and after.
agent: agent
tools: [search, read, edit, execute]
---

Refactor the selected code following `copilot/agents/refactorer.agent.md`.

**Contract:** behavior-preserving only. If you discover a bug, stop and
report it — do not fix it in this pass.

Work through:

1. **Diagnose** — in 2–4 sentences, what's wrong with the current shape
   and why it matters (readability, testability, fragility).

2. **Plan** — which refactor technique(s) apply (extract name, collapse
   conditional, eliminate flag parameter, push side effect to edge, etc.).
   One technique per pass when possible.

3. **Run the suite** — confirm it is green before touching anything.
   Quote the result.

4. **Apply** — make the behavior-preserving change.

5. **Run the suite again** — it must be green. Quote the result.

6. **Before / after** — show a brief summary of the shape change.

## Output format

```
## Diagnosis
<what's wrong and why it matters>

## Technique
<which refactor pattern, and why it fits>

## Before
<suite result before>

## After (diff summary)
<what changed structurally>

## After (suite)
<suite result after — must be identical pass count>

## Notes
<anything you noticed but did not change>
```
