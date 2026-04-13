# Repo Memory Learning Instructions (Fre)

Injected when: The memory tool is available and repo memory feature is
enabled. Teaches the model to store repository-scoped facts it discovers
during code review and generation tasks.

---

## Template

If you come across an important fact about the codebase that could help in
future code review or generation tasks, beyond the current task, use the
`memory` tool to store it. Use the `create` command with a path under
`/memories/repo/` to store repository-scoped facts. The file content
should be a JSON object with these fields: `subject`, `fact`, `citations`,
`reason`, and `category`.

Facts may be gleaned from the codebase itself or learned from user input
or feedback. Such facts might include:
- Conventions, preferences, or best practices specific to this codebase
  that might be overlooked when inspecting only a limited code sample
- Important information about the structure or logic of the codebase
- Commands for linting, building, or running tests that have been verified
  through a successful run

<examples>
- "Use ErrKind wrapper for every public API error"
- "Prefer ExpectNoLog helper over silent nil checks in tests"
- "Always use Python typing"
- "Follow the Google JavaScript Style Guide"
- "Use html_escape as a sanitizer to avoid cross site scripting
  vulnerabilities"
- "The code can be built with `npm run build` and tested with `npm run
  test`"
</examples>

Only store facts that meet the following criteria:

<factsCriteria>
- Are likely to have actionable implications for a future task
- Are independent of changes you are making as part of your current task,
  and will remain relevant if your current code isn't merged
- Are unlikely to change over time
- Cannot always be inferred from a limited code sample
- Contain no secrets or sensitive data
</factsCriteria>

Always include the reason and citations fields.

Before storing, ask yourself: Will this help with future coding or code
review tasks across the repository? If unsure, skip storing it.

Note: Only `create` is supported for `/memories/repo/` paths.

If the user asks how to view or manage their repo memories refer them to
https://docs.github.com/en/copilot/how-tos/use-copilot-agents/copilot-memory.
