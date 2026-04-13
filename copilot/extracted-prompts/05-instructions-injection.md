# Custom Instructions Injection (Ji Component)

This is how user-authored instructions are loaded and injected into the prompt.

---

## Loading Order

1. **Chat variable instructions** — inline context from #-references in the conversation
2. **Agent instructions** — files discovered by the customInstructionsService:
   - `.github/copilot-instructions.md`
   - `AGENTS.md` (root or nearest in directory tree for monorepos)
   - `.instructions.md` files matching `applyTo` patterns for current files
3. **VS Code setting-based instructions**:
   - `chat.codeGeneration.instructions`
   - `chat.testGeneration.instructions`
   - `chat.codeFeedback.instructions`
   - `chat.commitMessage.instructions`
   - `chat.pullRequestDescription.instructions`

## Deduplication

- URI-based: same file URI not loaded twice
- Content-based: identical text content not included twice

## Injected Preamble

When instructions are present, they are wrapped with this introduction:

> When generating code, please follow these user provided coding instructions.

For multi-root workspaces, an additional note:

> This is a multi-root workspace. The instructions below may come from
> different workspace folders. Apply each set of instructions to the folder
> it belongs to.

And a safety valve:

> You can ignore an instruction if it contradicts a system message.

## Placement

Instructions can be placed in either SystemMessage or UserMessage depending
on the `chat.customInstructionsInSystemMessage` VS Code setting. Default
behavior places them in UserMessage.

## XML Wrapping

Each instruction file is wrapped in:
```xml
<attachment filePath="path/to/file.instructions.md">
{file content}
</attachment>
```

The whole block is wrapped in:
```xml
<instructions>
...all instruction attachments...
</instructions>
```
