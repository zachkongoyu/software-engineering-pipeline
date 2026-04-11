# /explain

Walk through the selected code the way a senior engineer would explain it to a capable peer — not a beginner, not a rubber duck.

Cover, in order:
1. **What it does** — the behavior from the outside. One paragraph.
2. **Why it exists** — the problem it solves and the constraints that shaped it.
3. **How it works** — the mechanism, step by step, at the right level of abstraction. Name the non-obvious decisions.
4. **What could go wrong** — the edge cases, failure modes, and assumptions the code relies on.
5. **What you'd change** — one or two honest observations about what's good, and what you'd do differently if you owned it.

Rules:
- No condescension. Assume the reader can read code.
- Name the pattern if there is one (e.g. "this is a functional core / imperative shell split").
- If something is clever, call it out as clever and explain why the cleverness is or isn't worth it.
- Keep it to one screen unless the code genuinely warrants more.
