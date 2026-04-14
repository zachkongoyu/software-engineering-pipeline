# Conversation Compaction / Summary Prompt (XQn)

Used when the conversation exceeds the token budget and needs to be
compressed into a summary that preserves all essential context for
continuation.

## System Message (XQn constant)

```
Your task is to create a comprehensive, detailed summary of the entire conversation that captures all essential information needed to seamlessly continue the work without any loss of context. This summary will be used to compact the conversation while preserving critical technical details, decisions, and progress.

## Recent Context Analysis
Pay special attention to the most recent agent commands and tool executions that led to this summarization being triggered. Include:
- **Last Agent Commands**: What specific actions/tools were just executed
- **Tool Results**: Key outcomes from recent tool calls (truncate if very long, but preserve essential information)
- **Immediate State**: What was the system doing right before summarization
- **Triggering Context**: What caused the token budget to be exceeded

## Analysis Process
Before providing your final summary, wrap your analysis in `<analysis>` tags to organize your thoughts systematically:
1. **Chronological Review**: Go through the conversation chronologically, identifying key phases and transitions
2. **Intent Mapping**: Extract all explicit and implicit user requests, goals, and expectations
3. **Technical Inventory**: Catalog all technical concepts, tools, frameworks, and architectural decisions
4. **Code Archaeology**: Document all files, functions, and code patterns that were discussed or modified
5. **Progress Assessment**: Evaluate what has been completed vs. what remains pending
6. **Context Validation**: Ensure all critical information for continuation is captured
7. **Recent Commands Analysis**: Document the specific agent commands and tool results from the most recent operations

## Summary Structure
Your summary must include these sections in order:

<analysis>
[Chronological Review]
[Intent Mapping]
[Technical Inventory]
[Code Archaeology]
[Progress Assessment]
[Context Validation]
[Recent Commands Analysis]
</analysis>

<summary>
1. Conversation Overview:
- Primary Objectives: [All explicit user requests and overarching goals with exact quotes]
- Session Context: [High-level narrative of conversation flow and key phases]
- User Intent Evolution: [How user's needs or direction changed throughout conversation]

2. Technical Foundation:
- [Core Technology 1]: [Version/details and purpose]
- [Framework/Library 2]: [Configuration and usage context]
- [Architectural Pattern 3]: [Implementation approach and reasoning]
- [Environment Detail 4]: [Setup specifics and constraints]

3. Codebase Status:
- [File Name 1]:
  - Purpose: [Why this file is important to the project]
  - Current State: [Summary of recent changes or modifications]
  - Key Code Segments: [Important functions/classes with brief explanations]
  - Dependencies: [How this relates to other components]
- [File Name 2]:
  - Purpose: [Role in the project]
  - Current State: [Modification status]
  - Key Code Segments: [Critical code blocks]
- [Additional files as needed]

4. Problem Resolution:
- Issues Encountered: [Technical problems, bugs, or challenges faced]
- Solutions Implemented: [How problems were resolved and reasoning]
- Debugging Context: [Ongoing troubleshooting efforts or known issues]
- Lessons Learned: [Important insights or patterns discovered]

5. Progress Tracking:
- Completed Tasks: [What has been successfully implemented with status indicators]
- Partially Complete Work: [Tasks in progress with current completion status]
- Validated Outcomes: [Features or code confirmed working through testing]

6. Active Work State:
- Current Focus: [Precisely what was being worked on in most recent messages]
- Recent Context: [Detailed description of last few conversation exchanges]
- Working Code: [Code snippets being modified or discussed recently]
- Immediate Context: [Specific problem or feature being addressed before summary]

7. Recent Operations:
- Last Agent Commands: [Specific tools/actions executed just before summarization with exact command names]
- Tool Results Summary: [Key outcomes from recent tool executions - truncate long results but keep essential info]
- Pre-Summary State: [What the agent was actively doing when token budget was exceeded]
- Operation Context: [Why these specific commands were executed and their relationship to user goals]

8. Continuation Plan:
- [Pending Task 1]: [Details and specific next steps with verbatim quotes]
- [Pending Task 2]: [Requirements and continuation context]
- [Priority Information]: [Which tasks are most urgent or logically sequential]
- [Next Action]: [Immediate next step with direct quotes from recent messages]
</summary>

## Quality Guidelines
- **Precision**: Include exact filenames, function names, variable names, and technical terms
- **Completeness**: Capture all context needed to continue without re-reading the full conversation
- **Clarity**: Write for someone who needs to pick up exactly where the conversation left off
- **Verbatim Accuracy**: Use direct quotes for task specifications and recent work context
- **Technical Depth**: Include enough detail for complex technical decisions and code patterns
- **Logical Flow**: Present information in a way that builds understanding progressively

This summary should serve as a comprehensive handoff document that enables seamless continuation of all active work streams while preserving the full technical and contextual richness of the original conversation.
```

## User Message (sent after system prompt)

```
Summarize the conversation history so far, paying special attention to
the most recent agent commands and tool results that triggered this
summarization. Structure your summary using the enhanced format provided
in the system message.

[For Claude Opus only:]
IMPORTANT: Do NOT call any tools. Your only task is to generate a text
summary of the conversation. Do not attempt to execute any actions or
make any tool calls.

Focus particularly on:
- The specific agent commands/tools that were just executed
- The results returned from these recent tool calls (truncate if very long
  but preserve key information)
- What the agent was actively working on when the token budget was exceeded
- How these recent operations connect to the overall user goals

Include all important tool calls and their results as part of the
appropriate sections, with special emphasis on the most recent operations.
```

## Inline Summarization Mode (dRt)

When `inlineSummarization` is enabled (instead of `triggerSummarize`), the
model is asked to produce a summary mid-stream rather than in a separate
request:

```
The conversation has grown too large for the context window and must be
compacted now.

[...full XQn system prompt is repeated here...]

IMPORTANT: Output your summary wrapped in <summary> and </summary> tags.
Do NOT call any tools. Your ONLY task right now is to produce a comprehensive
summary of the conversation so far.

[For Claude Opus:]
IMPORTANT: Do NOT call any tools. Your only task is to generate a text
summary of the conversation. Do not attempt to execute any actions or
make any tool calls.
```

## Simple Mode (c$e)

When summarization fails or when forced via configuration
(`AgentHistorySummarizationMode = "simple"`), simple mode renders a
compressed version without asking the model to summarize:

```
The following is a compressed version of the preceeding history in the
current conversation. The first message is kept, some history may be
truncated after that:

<user>{first message}</user>
<conversation-summary>{any existing summary from prior compaction}</conversation-summary>
[...remaining rounds as <assistant>/<tool> with truncated results...]
```

- The first message is always fully preserved (priority: MAX_SAFE_INTEGER)
- Subsequent messages use a `PrioritizedList` (priority: 5000, descending: false)
  so older messages are trimmed first
- Tool arguments are truncated to 200 chars via `EX()`
- Tool results are truncated to `maxToolResultLength / 2`
- For GPT-4.1 models, `<reminderInstructions>` are injected after each
  existing summary block

## Transcript Fallback

After summarization, if a session transcript file exists, the summary
is augmented with a fallback path:

```
If you need specific details from before compaction (such as exact code
snippets, error messages, tool results, or content you previously
generated), use the read_file tool to look up the full uncompacted
conversation transcript at: "{transcriptPath}"
At the time of this request, the transcript has {lineCount} lines.
Example usage: read_file(filePath: "{transcriptPath}")
```

## Notes

- The `oEe` component wraps this in a `SystemMessage` and adds the
  conversation history as user/assistant message pairs for the model
  to summarize.
- For Claude Opus models (`claude-opus` prefix check), an extra
  "Do NOT call any tools" warning is added.
- The `simpleMode` flag toggles between compact (`c$e`) and full (`l$e`)
  history rendering.
- Additional summarization instructions can be appended from the
  `summarizationInstructions` prop.
- `getSummaryWithFallback()` tries "full" mode first, falls back to
  "simple" on error.
- The summarizer supports a `PreCompact` lifecycle hook that runs
  before compaction begins.
- After compaction, the summary replaces the old history and a
  `<conversation-summary>` tag wraps it in future prompts.
