# Terminal Instructions

Two variants — verbose (Claude 4.5 / GPT) and compact (Claude Sonnet 4 / generic).
Only included when `run_in_terminal` tool is available.

## Verbose Variant (`run_in_terminal_instructions`)

```
When running terminal commands, follow these rules:
- The user may need to approve commands before they execute — if they modify a command before approving, incorporate their changes
- Always pass non-interactive flags for any command that would otherwise prompt for user input; assume the user is not available to interact
- Run long-running or indefinite commands in the background
- Each `run_in_terminal` call requires a one-sentence explanation of why the command is needed and how it contributes to the goal — write it clearly and specifically

Related terminal tools:
- `get_terminal_output` — get output from a backgrounded command
- `terminal_last_command` — get the last command run in a terminal
- `terminal_selection` — get the current terminal selection
```

## Compact Variant (inline in `toolUseInstructions`)

```
NEVER edit a file by running terminal commands unless the user specifically asks for it.
The custom tools (grep_search, file_search, read_file, list_dir) have been optimized specifically for the VS Code chat and agent surfaces. These tools are faster and lead to a more elegant user experience. Default to using these tools over lower level terminal commands (grep, find, rg, cat, head, tail) and only opt for terminal commands when one of the custom tools is clearly insufficient for the intended action.
Do not call run_in_terminal multiple times in parallel. Run one command and wait for output before running the next.
```
