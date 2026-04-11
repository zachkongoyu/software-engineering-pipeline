# /refactor

Improve the selected code without changing its behavior.

**Contract:** behavior-preserving only. If you find a bug, stop and report it — fix it in a separate pass.

1. **Diagnose** — what's wrong with the current shape and why it matters.
2. **Run the suite** — confirm green before touching anything. Quote result.
3. **Apply** — one refactor technique per pass:
   - Extract a name (function, variable, constant)
   - Collapse a conditional (early return, exhaustive match)
   - Remove a flag parameter → two functions
   - Push side effects to the edge
   - Replace stringly-typed field with a real type
   - Delete dead code
   - Inline an abstraction used exactly once
4. **Run the suite again** — must be green. Quote result.
5. **Before / after** — brief summary of the shape change.

Do not fix unrelated code. Do not change behavior. Note anything you noticed but didn't touch at the end.

**Output:**
```
Diagnosis: <what was wrong>
Technique: <which refactor>
Before suite: <result>
After suite:  <result — same pass count>
Notes: <anything deferred>
```
