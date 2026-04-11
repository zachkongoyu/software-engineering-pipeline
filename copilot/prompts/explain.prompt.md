---
name: explain
description: Walk through selected code the way a senior engineer would explain it to a capable peer. Covers what it does, why it exists, how it works, what can go wrong, and what you'd change.
agent: agent
argument-hint: "Code or file to explain"
tools: ['search', 'read']
---

Walk through the selected code or `$FILE_OR_SYMBOL` the way a senior engineer explains it to a capable peer — not a beginner, not a rubber duck.

Cover these five things in order:

**1. What it does**
The behavior from the outside. One paragraph. What goes in, what comes out, what side effects it has.

**2. Why it exists**
The problem it solves and the constraints that shaped it. If there's an obvious alternative, name it and say why this was chosen instead.

**3. How it works**
The mechanism, step by step, at the right level of abstraction. Name the non-obvious decisions. If there's a recognizable pattern (functional core / imperative shell, state machine, builder, etc.), name it.

**4. What can go wrong**
The edge cases, failure modes, and assumptions the code silently relies on. What happens with empty input, concurrent callers, a network timeout, a missing config key.

**5. What you'd change**
One or two observations: what's well done and worth keeping, and what you'd do differently if you owned it. Be honest, not diplomatic.

Rules:
- No condescension. Assume the reader can read code.
- If something is clever, call it out and say whether the cleverness is worth it.
- Keep it to one screen unless the code genuinely warrants more.
- No bullet soup. Prose for 1–4, a short list only if 5 has multiple items.
