# VS Code Copilot Chat — System Prompt Assembly

> Reverse-engineered from `github.copilot-chat-0.43.0` extension bundle.
> Source: `C:\Users\user\.vscode\extensions\github.copilot-chat-0.43.0\dist\extension.js`

---

## Overview

The Copilot Chat system prompt is **not** a single static string. It is a
**component tree** rendered by a JSX-like framework (`vscpp()` / `vscppf`
fragments) with **priority-based token budgeting**. Components declare a
`priority` (higher = kept first when trimming) and `flexGrow` (how eagerly
they expand to fill the token budget). When the assembled prompt exceeds the
model's context window, lower-priority sections are dropped first.

---

## Architecture Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                     PromptRegistry (ks)                            │
│  Selects the right prompt variant based on model family            │
│  familyPrefixes: ["claude","gemini","gpt","gpt-5-codex",...]      │
└──────────────────┬──────────────────────────────────────────────────┘
                   │ resolveSystemPrompt(endpoint)
                   ▼
┌─────────────────────────────────────────────────────────────────────┐
│                  AgentPrompt (j4) — Top-Level Orchestrator         │
│  render() composes the full prompt tree from these sections:       │
│                                                                    │
│  ┌─── SystemMessage ──────────────────────────────────────────┐    │
│  │  1. Role identity     "You are an expert AI..."            │    │
│  │  2. CopilotIdentity   Name + model disclosure rules       │    │
│  │  3. SafetyRules        Microsoft content policies          │    │
│  └────────────────────────────────────────────────────────────┘    │
│                                                                    │
│  ┌─── SystemMessage (model-specific prompt) ──────────────────┐    │
│  │  4. Model-variant system prompt (see Model Variants below) │    │
│  │     - Parallel tool use instructions                       │    │
│  │     - Semantic search instructions                         │    │
│  │     - Replace-string instructions                          │    │
│  │     - Todo list instructions                               │    │
│  │     - Terminal instructions                                │    │
│  │     - Final answer formatting                              │    │
│  │     - File linkification rules                             │    │
│  │     - Code block formatting rules                          │    │
│  └────────────────────────────────────────────────────────────┘    │
│                                                                    │
│  ┌─── SystemMessage ──────────────────────────────────────────┐    │
│  │  5. Memory instructions (Fre)                              │    │
│  │     User memory, session memory, repo memory scopes        │    │
│  └────────────────────────────────────────────────────────────┘    │
│                                                                    │
│  ┌─── SystemMessage or UserMessage ──────────────────────────┐     │
│  │  6. Custom Instructions (Ji)                               │    │
│  │     a. copilot-instructions.md / AGENTS.md                 │    │
│  │     b. .instructions.md files (applyTo-matched)            │    │
│  │     c. VS Code settings-based instructions                 │    │
│  │     d. Skill bodies (on-demand loaded SKILL.md)            │    │
│  │                                                            │    │
│  │  7. Mode Instructions (if custom agent mode active)        │    │
│  │     "You are currently running in X mode..."               │    │
│  │     Body of the .agent.md file                             │    │
│  └────────────────────────────────────────────────────────────┘    │
│                                                                    │
│  ┌─── SystemMessage (autopilot only) ────────────────────────┐     │
│  │  8. task_complete instructions (priority: 80)              │    │
│  └────────────────────────────────────────────────────────────┘    │
│                                                                    │
│  ┌─── SystemMessage ──────────────────────────────────────────┐    │
│  │  9. Template variables context (if any)                    │    │
│  └────────────────────────────────────────────────────────────┘    │
│                                                                    │
│  ┌─── UserMessage ────────────────────────────────────────────┐    │
│  │  10. Global Agent Context (s$e)                            │    │
│  │      - Environment info (OS, date)                         │    │
│  │      - Workspace structure                                 │    │
│  │      - User/session/repo memory contents                   │    │
│  │      - Editor context (active file)                        │    │
│  │      - Terminal state                                      │    │
│  └────────────────────────────────────────────────────────────┘    │
│                                                                    │
│  ┌─── Conversation History (P7e / R8) ────────────────────────┐    │
│  │  11. Previous turns, tool results, assistant responses     │    │
│  │      (priority: 700, flexGrow: 1)                          │    │
│  └────────────────────────────────────────────────────────────┘    │
│                                                                    │
│  ┌─── UserMessage (current turn) (B8) ───────────────────────┐    │
│  │  12. Tool references / attachments (priority: 899)         │    │
│  │  13. Chat variables context (priority: 750)                │    │
│  │  14. Tool results (fetched content) (priority: 899)        │    │
│  │  15. User's actual message (priority: 900)                 │    │
│  │  16. Reminder instructions (end-of-prompt nudge)           │    │
│  └────────────────────────────────────────────────────────────┘    │
│                                                                    │
│  ┌─── Tool Call Rounds (Xc) ─────────────────────────────────┐     │
│  │  17. Previous tool call/result pairs in this turn          │    │
│  │      (priority: 899, flexGrow: 2)                          │    │
│  └────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Priority Map

Higher priority = kept when token budget is tight.

| Priority | Section                                  |
| -------- | ---------------------------------------- |
| 1000     | Inline system instructions (Panel mode)  |
| 900      | User message, chat variables context     |
| 899      | Tool attachments, tool call rounds       |
| 800      | Date/environment context                 |
| 750      | Custom instructions (Ji)                 |
| 700      | Conversation history                     |
| 600      | Additional context (open editors, etc.)  |
| 80       | Autopilot task_complete instructions     |

---

## Model-Family → Prompt Variant Mapping

The `PromptRegistry` (`ks`) selects a prompt variant based on the model
being used. Each variant resolves a different `SystemPrompt`, `ReminderInstructions`,
`CopilotIdentityRules`, and `SafetyRules` class.

### Complete 22-Prompt Registry (ks.registerPrompt calls)

| # | Registry Class | Family Prefixes / Matcher | System Prompt | Reminder | File |
|---|----------------|---------------------------|---------------|----------|------|
| 1 | `cBt` | `["claude","Anthropic"]` | Sonnet4→`oBt`, 4.5→`jxe`, Opus→`sBt`, else→`aBt` | Sonnet4/4.5→`Wxe`, else→`Hxe` | `claude-variants.md`, `claude-sonnet4.md`, `claude-4.5.md` |
| 2 | `dBt` | `["gemini"]` | `dBt` (default) | `mBt` | `gemini.md` |
| 3 | `pBt` | `["gemini"]` + experiment flag | `pBt` (alternate) | `mBt` | `gemini.md` |
| 4 | `IBt` | `["gpt","o4-mini","o3-mini","OpenAI"]` | `IBt` | `SBt` | `gpt-family.md` |
| 5 | `kBt` | `["gpt","o4-mini","o3-mini","OpenAI"]` + experiment | `kBt` | `SBt` | `gpt-family.md` |
| 6 | `HBt` | `["gpt-5-codex","gpt-5.1-preview-codex","gpt-5.1-mini-codex"]` | `HBt` | `SBt` | `gpt5-codex.md` |
| 7 | `WBt` | same as HBt + experiment flag | `WBt` | `SBt` | `gpt5-codex.md` |
| 8 | `TBt` | `matchesModel()` — hashed codex agent families | `TBt` (codex editing) | `Vxe` | `gpt5x-coding-agents.md` |
| 9 | `RBt` | `matchesModel()` — hashed coding agent set 1 | `RBt` (full coding agent) | `MBt` | `gpt5x-coding-agents.md` |
| 10 | `NBt` | `matchesModel()` — hashed coding agent set 2 | `NBt` (=`RBt` template) | `OBt` | `gpt5x-coding-agents.md` |
| 11 | `ZBt` | `matchesModel()` — hashed coding agent set 3 | `ZBt` (=`RBt` template) | `jBt` | `gpt5x-coding-agents.md` |
| 12 | `JBt` | `matchesModel()` — hashed coding agent set 4 | `JBt` (=`RBt` template) | `i$e` | `gpt5x-coding-agents.md` |
| 13 | `Yxe` | `matchesModel()` — hashed coding agent set 5 | `Yxe` (=`RBt` template) | `o$e` | `gpt5x-coding-agents.md` |
| 14 | `z4` | `matchesModel()` — GPT-5.4 large prompt | `z4` (hypothesis-driven) | `o$e` | `gpt5x-coding-agents.md` |
| 15 | `$4` | `matchesModel()` — GPT-5.4 concise prompt | `$4` (engineering values) | `o$e` | `gpt5x-coding-agents.md` |
| 16 | `zBt` | `matchesModel()` — GPT-5.4 default | `zBt` (=`RBt` fallback) | `o$e` | `gpt5x-coding-agents.md` |
| 17 | `nRt` | `matchesModel()` — grok-code hashed | `nRt` | None | `grok-code.md` |
| 18 | `iRt` | `matchesModel()` — `/glm[-_]?4[._p]?[67]/` regex | `iRt` | `kI` | `glm4.md` |
| 19 | `ABt` | `matchesModel()` — Minimax hashed | `ABt` | `kI` | `internal-models.md` |
| 20 | `fBt` | `["vscModelA"]` | `gBt` (=`lBt` generic) | `n$e` | `internal-models.md` |
| 21 | `vBt`/`_Bt`/`wBt` | `["vscModelB"]`/`["vscModelC"]` | `hBt`/`bBt` (=`lBt`) | `xBt`/`EBt` | `internal-models.md` |
| 22 | `CBt` | `["vscModelD"]` | `yBt` (alternate parallel) | `n$e` | `internal-models.md` |
| — | `lBt` | *(fallback — no match)* | `lBt` (generic) | `kI` | `generic-fallback.md` |

### Built-in Agent Prompts (dynamically generated .agent.md files)

| Agent | Class | Prompt File |
|-------|-------|-------------|
| Edit | `wL` (Fbi config) | `edit-mode-agent.md` |
| Ask | `TR` | `ask-agent.md` |
| Explore | `BR` | `explore-subagent.md` |
| Plan | (built-in) | `plan-mode.md` |

### Claude Sub-Variants

The Claude prompt resolver further differentiates:
- **Sonnet 4** → `oBt` (tuned for Sonnet 4)
- **Claude 4.5** → `jxe` (tuned for 4.5)
- **Opus** → `sBt` (tuned for Opus)
- **Other Claude** → `aBt` (default Claude)

---

## Shared Prompt Components (Minified Names → Purpose)

| Symbol | Purpose                                                              |
| ------ | -------------------------------------------------------------------- |
| `uo`   | **CopilotIdentity** — "Your name is GitHub Copilot. Using model X." |
| `yg`   | **CopilotIdentity (alt)** — shorter variant for some model families |
| `Vr`   | **SafetyRules** — Microsoft content policies, copyright, refusal    |
| `Wb`   | **SafetyRules (alt)** — shorter variant for some families           |
| `CI`   | **CodeBlockFormatting** — 4-backtick fences, filepath comments      |
| `Nn`   | **ExistingCodeMarker** — `...existing code...` ellipsis constant    |
| `Fre`  | **MemoryInstructions** — user/session/repo memory scopes            |
| `Ji`   | **CustomInstructions** — loads copilot-instructions.md + .instructions.md |
| `en`   | **SmartMessage** — routes to SystemMessage or UserMessage by model  |
| `q`    | **NamedSection** — wraps content in `<sectionName>` tags            |
| `vi`   | **ToolAttachments** — renders #-referenced tools and their output   |
| `Ta`   | **ChatVariablesContext** — renders attached files, selections       |
| `Ki`   | **ConversationContainer** — wraps history with priorities           |
| `j4`   | **AgentPrompt** — top-level orchestrator component                  |
| `R8`   | **ConversationHistory (cached)** — with cache breakpoints           |
| `P7e`  | **ConversationHistory (simple)** — without cache breakpoints        |
| `B8`   | **CurrentTurnMessage** — the current user message + context         |
| `Xc`   | **ToolCallRounds** — previous tool call/result pairs                |
| `T8`   | **AgentPersistenceReminder** — "keep going until resolved"          |
| `kI`   | **ReminderInstructions (default)** — end-of-prompt reminders        |

---

## Custom Instructions Loading (Ji Component)

The `Ji` component loads instructions from multiple sources and merges them:

```
1. Chat Variables (inline #-references)
   └── Variables marked as instructions in conversation context

2. Agent Instructions (customInstructionsService.getAgentInstructions())
   └── .github/copilot-instructions.md
   └── AGENTS.md (root or nearest in directory tree)
   └── .instructions.md files matching current applyTo patterns

3. VS Code Settings-Based Instructions
   └── chat.codeGeneration.instructions
   └── chat.testGeneration.instructions
   └── chat.codeFeedback.instructions
   └── chat.commitMessage.instructions
   └── chat.pullRequestDescription.instructions

4. Deduplication
   └── URI-based (same file not loaded twice)
   └── Content-based (identical text not included twice)
```

The introduction text before instructions:
> "When generating code, please follow these user provided coding instructions."

For multi-root workspaces:
> "This is a multi-root workspace. The instructions below may come from
> different workspace folders. Apply each set of instructions to the folder
> it belongs to."

---

## Mode Instructions Injection

When a custom `.agent.md` is active, its body is injected as:

```
<modeInstructions>
You are currently running in "{agentName}" mode. Below are your
instructions for this mode, they must take precedence over any
instructions above.

{content of the .agent.md body}
</modeInstructions>
```

---

## Named Section Tags

The prompt uses XML-like named sections (`<sectionName>`) for structure.
The complete catalog of known section names (from the section registry):

### System Instructions Category
`modeInstructions`, `reminderInstructions`, `notebookInstructions`,
`notebookFormatInstructions`, `fileLinkification`,
`replaceStringInstructions`, `applyPatchInstructions`,
`codebaseToolInstructions`, `codeSearchInstructions`,
`codeSearchToolUseInstructions`, `vscodeAPIToolUseInstructions`,
`vscodeCmdToolUseInstructions`, `searchExtensionToolUseInstructions`,
`extensionSearchResponseRules`, `grounding`, `gptAgentInstructions`,
`coding_agent_instructions`, `workflowGuidance`, `structuredWorkflow`,
`taskTracking`, `planning`, `planning_instructions`, `task_execution`,
`testing`, `validating_work`, `progress_updates`, `personality`,
`communicationStyle`, `communicationExamples`, `communicationGuidelines`,
`ambition_vs_precision`, `autonomy_and_persistence`, `contextManagement`,
`final_answer_formatting`, `final_answer_instructions`,
`final_answer_structure_and_style_guidelines`,
`presenting_your_work_and_final_message`, `tool_preambles`, `tool_use`,
`parallel_tool_use_instructions`, `editing_constraints`,
`exploration_and_reading_files`, `additional_notes`,
`handling_errors_and_unexpected_outputs`, `special_user_requests`,
`frontend_tasks`, `special_formatting`, `user_updates_spec`,
`design_and_scope_constraints`, `long_context_handling`,
`uncertainty_and_ambiguity`, `high_risk_self_check`, `patchFormat`,
`responseTemplate`, `importantReminders`, `memoryInstructions`,
`memoryScopes`, `memoryGuidelines`

### User Context — Files
`attachment`, `attachments`, `file`, `editorContext`, `currentDocument`,
`currentFile`, `resource`, `selection`, `documentFragment`,
`languageServerContext`, `symbolDefinitions`, `symbol`, `codeToTest`,
`testsFile`, `testExample`, `testDependencies`, `sampleTest`,
`relatedTest`, `relatedSource`, `readme`, `original-code`,
`code-changes`, `changeDescription`, `currentChange`, `cell`,
`cellsAbove`, `cellsBelow`, `cell-output`, `notebook-cell-output`,
`some_of_the_cells_after_edit`, `workspaceFolder`, `projectLabels`

### User Context — Tool Results
`error`, `errors`, `compileError`, `suggestedFix`, `testFailure`,
`cell-execution-error`, `stackFrame`, `feedback`, `analysis`,
`criteria`, `invalidPatch`, `correctedEdit`, `actualOutput`,
`expectedOutput`, `match`

### User Context — Messages
`userRequest`, `UserRequest`, `userPrompt`, `user_query`, `prompt`,
`context`, `environment_info`, `workspace_info`, `reminder`, `note`,
`todoList`, `task`, `toolReferences`, `user`, `assistant`, `tool`,
`conversation-summary`, `message`, `summary`, `example`, `examples`,
`response`, `Response`, `request`, `settings`, `command`,
`currentVSCodeVersion`, `releaseNotes`

---

## Prompt Constants

```javascript
Rbn = ".github/copilot-instructions.md"   // Legacy instructions path
_L  = ".instructions.md"                  // Instructions file extension
Vv  = ".agent.md"                         // Agent file extension
KOe = "SKILL.md"                          // Skill file name
ayt = "chat.instructionsFilesLocations"   // Instructions folders setting
mve = "chat.agentSkillsLocations"         // Skills folders setting
Sbn = [".github/skills", ".claude/skills"] // Default skill search paths
Tbn = [".copilot/skills", ".claude/skills"] // Alt skill search paths
Nn  = "...existing code..."               // Ellipsis marker in code blocks
M4  = "filepath:"                         // Filepath marker in code blocks
```

---

## File Locations

### Extension Bundle
```
C:\Users\user\.vscode\extensions\github.copilot-chat-0.43.0\
├── dist\extension.js          ← All prompt logic (minified, ~50MB)
├── assets\prompts\            ← Built-in prompt files and skills
│   ├── init.prompt.md
│   ├── plan.prompt.md
│   ├── create-agent.prompt.md
│   ├── create-hook.prompt.md
│   ├── create-instructions.prompt.md
│   ├── create-prompt.prompt.md
│   ├── create-skill.prompt.md
│   └── skills\
│       ├── agent-customization\SKILL.md + references\*.md
│       ├── get-search-view-results\SKILL.md
│       ├── install-vscode-extension\SKILL.md
│       ├── project-setup-info-context7\SKILL.md
│       ├── project-setup-info-local\SKILL.md
│       └── troubleshoot\SKILL.md
```

### Debug Logs (per session)
```
%APPDATA%\Code\User\workspaceStorage\<id>\GitHub.copilot-chat\debug-logs\<sessionId>\
├── main.jsonl                 ← Full conversation trace
├── system_prompt_0.json       ← Untruncated system prompt (!)
├── tools_0.json               ← Tool definitions sent to model
├── models.json                ← Available models snapshot
└── runSubagent-*.jsonl        ← Subagent traces
```

**The `system_prompt_0.json` file is the most direct way to see the
fully-assembled prompt for any session.** It contains the rendered output
after all components have been composed and before token truncation.

### Customization Files (User-Authored)
```
.github/copilot-instructions.md     ← Workspace instructions (always loaded)
.github/instructions/*.instructions.md  ← File-specific instructions
.github/agents/*.agent.md           ← Custom agents
.github/skills/<name>/SKILL.md      ← Custom skills
.github/prompts/*.prompt.md         ← Reusable prompts
.github/hooks/*.json                ← Lifecycle hooks

~/.copilot/instructions/             ← User-level instructions
~/.copilot/skills/<name>/            ← User-level skills
%APPDATA%\Code\User\prompts\         ← User-level prompts
```

---

## End-to-End Assembly Flow

```
1. REQUEST ARRIVES
   User sends message in chat

2. MODEL RESOLUTION
   PromptRegistry.getPromptResolver(endpoint)
   → Matches model family → selects prompt variant class

3. SYSTEM PROMPT RENDER
   AgentPrompt.render()
   ├── SystemMessage: Role + Identity + Safety
   ├── Model-specific prompt (tool instructions, formatting, etc.)
   ├── Memory instructions
   ├── Custom instructions (Ji)
   │   ├── Scan copilot-instructions.md / AGENTS.md
   │   ├── Match .instructions.md by applyTo globs
   │   ├── Load VS Code setting-based instructions
   │   └── Deduplicate by URI and content
   ├── Mode instructions (if .agent.md active)
   └── Template variables

4. USER MESSAGE RENDER
   ├── Global agent context (env, workspace, memory, editor)
   ├── Conversation history (previous turns)
   ├── Current turn attachments and tool results
   ├── User's actual message
   └── Reminder / nudge instructions

5. TOKEN BUDGETING
   Components have priority + flexGrow
   Framework trims lowest-priority sections first
   Target: modelMaxPromptTokens * 0.5 for tool results

6. SERIALIZATION
   Rendered tree → messages array → API call
   Written to system_prompt_0.json in debug logs
```

---

## Token Budgeting & Priority Trimming

The prompt renderer implements a **CSS-flexbox-inspired token allocation**
system. Each component declares layout hints:

| Prop | Meaning |
|------|---------|
| `priority` | Retention priority. Lower = trimmed first when over budget |
| `flexGrow` | Growth order. Lower value = rendered earlier, gets first claim on tokens. Default `Infinity` (render last) |
| `flexReserve` | Token count reserved for lower-`flexGrow` components. Can be absolute (`500`) or fractional (`"/3"` = 1/3 of remaining) |
| `flexBasis` | Relative share when multiple components share the same `flexGrow` level |
| `passPriority` | If false, child priorities are self-contained |

### Rendering Algorithm (`_processPromptPieces`)

```
1. SORT by flexGrow (ascending — lower grows first)
2. For each flexGrow level:
   a. Calculate RESERVES for all later-level components
   b. Consume reserved tokens from remaining budget
   c. Distribute remaining budget across components at this level
      proportional to flexBasis (default 1)
   d. Call prepare() then render() with per-component tokenBudget
   e. Process children recursively
   f. Release reserves back to pool
3. Result: a materialized tree of messages/chunks with token counts
```

### Over-Budget Trimming (`_getFinalElementTree`)

After the tree is fully materialized:

```
1. Compute total tokenCount
2. While tokenCount > limit:
   a. Find the lowest-priority leaf node (s_t)
      - Walk all children depth-first
      - Track ties by comparing nested priority (lowestNested)
   b. Remove that node and subtract its upperBoundTokenCount × 1.25
   c. Recompute actual tokenCount
3. If removal causes BudgetExceededError (no removable nodes left),
   the error carries the partial messages + metadata for debugging
```

### Growable Sections (Expandable)

Components using `<Expandable>` are re-rendered with surplus tokens:

```
1. After initial render + trimming, check if tokenCount < budget
2. For each Expandable component:
   a. Re-render with tokenBudget = (surplus + initialConsume)
   b. Replace old node with new materialized tree
   c. If new total >= budget, stop growing
```

### TokenLimit Scoping

`<TokenLimit max={N}>` creates a nested budget scope. A component inside
a `TokenLimit` is trimmed independently against that local cap, not just
the global model budget. Used for tool results and conversation history.

---

## Message Routing (`en` / InstructionMessage)

The `en` class routes instructions to the correct message role based on
model family:

```javascript
// d_n(family) returns true for "claude-3.5-sonnet"
render() {
  if (d_n(this.promptEndpoint.family))
    return <UserMessage>{children}</UserMessage>;   // Sonnet 3.5 → User
  else
    return <SystemMessage>{children}</SystemMessage>; // Everything else → System
}
```

**Why**: Claude 3.5 Sonnet performs better when instructions are in
the user role. All other models (GPT, Gemini, other Claude versions)
receive instructions in the system role.

---

## History Ordering (`Ki` / HistoryWithInstructions)

The `Ki` component controls whether conversation history appears
**before** or **after** the instruction messages:

```javascript
// R9(family) returns true for "claude-3.5-sonnet"
render() {
  const historyFirst = R9(this.promptEndpoint.family);
  return <>
    {historyFirst ? <History .../> : null}
    {...instructionChildren}
    {historyFirst ? null : <History .../>}
  </>;
}
```

| Model Family | History Position |
|---|---|
| Claude 3.5 Sonnet | **Before** instructions |
| All others | **After** instructions |

---

## Cache Breakpoints

Cache breakpoints are `<cacheBreakpoint type="ephemeral">` elements
that tell Anthropic's API where to cache prompt prefixes. Placement:

```
┌─ System Message ──────────────────────┐
│  Role identity + Safety + Instructions│
│  Model-specific prompt                │
│  Custom instructions + Context        │
│  <cacheBreakpoint type="ephemeral">   │  ← Breakpoint 1: after system+context
└───────────────────────────────────────┘

┌─ Tool Call Rounds ────────────────────┐
│  Round 1: tool calls + results        │
│  Round 2: tool calls + results        │
│  Round N (last): tool calls + results │
│  <cacheBreakpoint type="ephemeral">   │  ← Breakpoint 2: after last round
└───────────────────────────────────────┘
```

- Only placed when `enableCacheBreakpoints` is true (R8 history mode)
- The second breakpoint is conditional: `isLast && enableCacheBreakpoints`
- P7e (simple history mode) never uses cache breakpoints
- The cache type is always `"ephemeral"` (`o_` constant)

---

## Conversation Compaction (Summarization)

When the conversation grows too large, the `iEe` (ConversationHistorySummarizer)
compresses history into a summary. Two modes:

### Full Mode (`l$e`)
Uses the complete XQn compaction prompt (see `conversation-compaction.md`).
The model receives the full conversation history and produces a structured
summary with 8 sections (overview, technical foundation, codebase status,
problem resolution, progress tracking, active work state, recent operations,
continuation plan).

### Simple Mode (`c$e`)
A token-priority-based compression without model summarization:
- First message always kept at max priority
- Remaining messages in a `PrioritizedList` (oldest trimmed first)
- Tool arguments truncated to 200 chars
- Tool results truncated to `maxToolResultLength / 2`

### Summarization Pipeline

```
1. Check config: if AgentHistorySummarizationMode == "simple" → simple
2. Otherwise, try "full" mode
3. If full mode fails → fallback to "simple"
4. Execute PreCompact lifecycle hook (if registered)
5. Render the summarization prompt (oEe) against the endpoint
6. Send to model, get summary text
7. Append transcript fallback path (if transcript service active):
   "use read_file to look up the full uncompacted transcript at: {path}"
8. Replace compacted history with <conversation-summary>{summary}</conversation-summary>
```

### Inline vs Triggered Summarization

| Mode | Trigger | Mechanism |
|---|---|---|
| `triggerSummarize` | Separate summarization request | Model produces summary in a dedicated call; result replaces history |
| `inlineSummarization` | Mid-stream injection | A UserMessage is injected telling the model to output `<summary>...</summary>` tags inline |

### Transcript Storage

The `sessionTranscriptService` persists the full uncompacted conversation
to a file. After compaction, the summary includes a fallback:

```
If you need specific details from before compaction, use the read_file
tool to look up the full uncompacted conversation transcript at: "{path}"
At the time of this request, the transcript has {N} lines.
Example usage: read_file(filePath: "{path}")
```

---

## Opaque Content Parts

The renderer supports opaque (non-text) content parts via `<opaque>` elements.
Five types exist:

| Type (`mc` enum) | Purpose |
|---|---|
| `cache_control` | Cache breakpoint markers (see above) |
| `stateful_marker` | Server-side session state tracking. `invalid_stateful_marker` → "Your chat session state is invalid, please start a new chat." |
| `thinking` | Model reasoning/thinking data. Rendered by `dte` class with `tokenUsage` tracking |
| `context_management` | Anthropic server-side context editing. Reports `cleared_input_tokens`, `cleared_tool_uses`, `cleared_thinking_turns` |
| `phase_data` | Internal phase tracking metadata |

---

## Anthropic-Specific Request Headers

For Claude models (Messages API, request types 7/9), extra headers are
conditionally added:

```
anthropic-beta: [comma-separated list]
  - "interleaved-thinking-2025-05-14"    (if !supportsAdaptiveThinking)
  - "context-management-2025-06-27"      (if n1() config check passes)
  - "advanced-tool-use-2025-11-20"       (if Um() config check passes)

X-Model-Provider-Preference: {value}     (if TeamInternal.ModelProviderPreference set)
```

---

## Model Capability Flags

The `PromptEndpoint` class tracks capabilities that affect prompt assembly:

| Flag | Effect |
|---|---|
| `supportsAdaptiveThinking` | If false, adds `interleaved-thinking-2025-05-14` beta header |
| `minThinkingBudget` / `maxThinkingBudget` | Bounds for thinking token allocation |
| `supportsReasoningEffort` | Whether the model supports `reasoning_effort` parameter |
| `thinking_budget` | Token budget for model thinking/reasoning |
| `maxPromptImages` | Limits vision content in prompts |
| `useMessagesApi` | Anthropic Messages API vs OpenAI Completions API |
| `useResponsesApi` | OpenAI Responses API (newer) |
| `customModel` | Whether this is a user-contributed model |

---

## Materialized Tree Types

The rendering pipeline produces a tree of these node types:

| Class | Role |
|---|---|
| `UL` (GenericMaterializedContainer) | Container node with children, priority, flags |
| `e1` (MaterializedChatMessage) | A single chat message (role, name, toolCalls, toolCallId) |
| `V_e` (MaterializedChatMessageTextChunk) | A text fragment with priority |
| `XS` (MaterializedChatMessageBreakpoint) | A cache breakpoint marker |
| `Xz` (MaterializedChatMessageImage) | An image content part |
| `ej` (MaterializedChatMessageDocument) | A document content part |
| `Z9` (MaterializedChatMessageOpaque) | An opaque content part (thinking, etc.) |

### Container Flags

| Flag (bitmask) | Meaning |
|---|---|
| `1` | `ForceRemoval` — children can be force-removed during trimming |
| `2` | `Atomic` — remove entire node, don't recurse into children |
| `4` | `PassPriority` — container is transparent for priority traversal |
| `8` | `EmptyAlternate` — has exactly 2 children; use second if non-empty, else first |

---

## Prompt Element Primitives

| Component | Purpose |
|---|---|
| `SystemMessage` (`C8e`) | System-role message wrapper |
| `UserMessage` (`x8e`) | User-role message wrapper |
| `AssistantMessage` (`E8e`) | Assistant-role message wrapper |
| `ToolMessage` (`d_t`) | Tool-role message wrapper |
| `TextChunk` (`I8e`) | Text with `breakOn` / `breakOnWhitespace` for token-aware splitting |
| `PrioritizedList` (`A_t`) | Auto-assigns descending/ascending priorities to children |
| `TokenLimit` (`y_t`) | Scoped token budget cap |
| `Expandable` (`b_t`) | Lazy-rendered section that grows with surplus budget |
| `IfEmpty` (`v_t`) | Shows alt content when primary is empty (uses `EmptyAlternate` flag) |
| `Chunk` (`h_t`) | Generic grouping container |
| `ToolResult` (`f_t`) | Renders tool result content |
| `LegacyPrioritization` (`g_t`) | Backward-compat priority wrapper |
