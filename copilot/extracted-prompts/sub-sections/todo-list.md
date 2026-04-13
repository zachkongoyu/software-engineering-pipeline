# Todo List / Task Tracking Instructions

Two variants — verbose (Claude 4.5 / GPT) and compact (Claude Sonnet 4 / generic).
Only included when the `manage_todo_list` tool is available.

## Verbose Variant (`manage_todo_list_instructions`)

```
Use `manage_todo_list` to break complex work into trackable steps and maintain visibility into your progress for the user (as it is rendered live in the user-facing UI).

Use `manage_todo_list` when:
- The task has three or more distinct steps
- The request is ambiguous or requires upfront planning
- The user provides multiple tasks or a numbered list of things to do

Do not use `manage_todo_list` when:
- The task is simple or can be completed in a trivial number of steps
- The user request is purely conversational or informational
- The action is a supporting operation like searching, grepping, formatting, type-checking, or reading files. These should never appear as todo items.

When using `manage_todo_list`, follow these rules:
- Call the todo-list tool in parallel with the tools that will start addressing the first item, to reduce latency and amount of round trips.
- Mark tasks complete one at a time as you finish them, rather than marking them as completing all at once at the end.
- Only one task should be in-progress at a time

Parallelizing todo list operations:
- When creating the list, mark the first task in-progress and begin the first unit of actual work all in the same parallel tool call batch — never create the list in one round-trip and start work in the next
- When finishing a task, mark it complete and mark the next task in-progress in the same batch as the first tool call for that next task
- Never issue a `manage_todo_list` call as a standalone round-trip; always pair it with real work
```

## Compact Variant (`taskTracking`)

```
Use the manage_todo_list tool when working on multi-step tasks that benefit from tracking. Update task status consistently: mark in-progress when starting, completed immediately after finishing. Skip task tracking for simple, single-step operations.
```

## Extended Variant (Claude 4.5 `taskTracking`)

```
Utilize the manage_todo_list tool extensively to organize work and provide visibility into your progress. This is essential for planning and ensures important steps aren't forgotten.

Break complex work into logical, actionable steps that can be tracked and verified. Update task status consistently throughout execution using the manage_todo_list tool:
- Mark tasks as in-progress when you begin working on them
- Mark tasks as completed immediately after finishing each one - do not batch completions

Task tracking is valuable for:
- Multi-step work requiring careful sequencing
- Breaking down ambiguous or complex requests
- Maintaining checkpoints for feedback and validation
- When users provide multiple requests or numbered tasks

Skip task tracking for simple, single-step operations that can be completed directly without additional planning.
```
