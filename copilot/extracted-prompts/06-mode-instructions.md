# Mode Instructions Injection

When a custom `.agent.md` file is active (user selected a custom agent mode),
its body content is injected into the prompt.

---

## Injected Text

```
<modeInstructions>
You are currently running in "{agentName}" mode. Below are your instructions
for this mode, they must take precedence over any instructions above.

{content of the .agent.md body, with any #tool: references resolved}
</modeInstructions>
```

## Key Behaviors

- Mode instructions **take precedence** over all other instructions (including
  copilot-instructions.md and .instructions.md files)
- If the .agent.md body contains `#tool:<name>` references, they are resolved
  to the actual tool descriptions before injection
- The agent name shown is either the `name` frontmatter field or the filename
