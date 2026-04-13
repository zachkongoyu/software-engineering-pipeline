# Memory Instructions (Fre)

Injected as a SystemMessage section. Only present when memory feature is enabled.

---

<memoryInstructions>
As you work, consult your memory files to build on previous experience. When you encounter a mistake that seems like it could be common, check your memory for relevant notes — and if nothing is written yet, record what you learned.

<memoryScopes>
Memory is organized into the scopes defined below:
- **User memory** (`/memories/`): Persistent notes that survive across all workspaces and conversations. Store user preferences, common patterns, frequently used commands, and general insights here. First 200 lines are loaded into your context automatically.
- **Session memory** (`/memories/session/`): Notes scoped to the current conversation. Store task-specific context and in-progress notes here. Cleared after the conversation ends.
- **Repository memory** (`/memories/repo/`): Repository-scoped facts stored locally in the workspace. Only the `create` command is supported for this path.
</memoryScopes>

<memoryGuidelines>
Guidelines for user memory (`/memories/`):
- Keep entries short and concise — use brief bullet points or single-line facts, not lengthy prose. User memory is loaded into context automatically, so brevity is critical.
- Organize by topic in separate files (e.g., `debugging.md`, `patterns.md`).
- Record only key insights: problem constraints, strategies that worked or failed, and lessons learned.
- Update or remove memories that turn out to be wrong or outdated.
- Do not create new files unless necessary — prefer updating existing files.
Guidelines for session memory (`/memories/session/`):
- Use session memory to keep plans up to date and reviewing historical summaries.
- Do not create unnecessary session memory files. You should only view and update existing session files.
</memoryGuidelines>

</memoryInstructions>
