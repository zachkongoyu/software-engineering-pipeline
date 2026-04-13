# Extracted System Prompts — VS Code Copilot Chat 0.43.0

This directory contains system prompts reverse-engineered from the Copilot
Chat extension bundle. Each file represents a different layer or variant
of the prompt assembly pipeline.

## Directory Structure

```
extracted-prompts/
├── README.md                     ← This file
├── 01-role-identity.md           ← "You are an expert AI..." (shared header)
├── 02-copilot-identity.md        ← Name / model disclosure rules
├── 03-safety-rules.md            ← Microsoft content policies
├── 04-memory-instructions.md     ← User/session/repo memory system
├── 05-instructions-injection.md  ← How copilot-instructions.md is loaded
├── 06-mode-instructions.md       ← How .agent.md modes are injected
├── model-variants/
│   ├── claude-sonnet4.md         ← Claude Sonnet 4 (oBt) system prompt
│   ├── claude-4.5.md             ← Claude 4.5 (jxe) system prompt
│   ├── claude-variants.md        ← Claude architecture: gie base + aBt/sBt + reminders
│   ├── gpt-family.md             ← GPT / OpenAI family prompt (IBt/kBt)
│   ├── gpt5-codex.md             ← GPT-5 Codex prompt (HBt/WBt)
│   ├── gpt5x-coding-agents.md   ← GPT-5.x coding agents (TBt/RBt/NBt/ZBt/JBt/z4/$4/zBt)
│   ├── gemini.md                 ← Gemini family (dBt/pBt) with grounding
│   ├── grok-code.md              ← Grok-code family (nRt)
│   ├── glm4.md                   ← GLM-4 family (iRt)
│   ├── internal-models.md        ← Internal/experimental models (ABt/gBt/hBt/bBt/yBt)
│   ├── generic-fallback.md       ← Default fallback prompt (lBt)
│   ├── edit-mode.md              ← Edit Mode restricted agent
│   ├── ask-mode.md               ← Ask Mode read-only agent
│   └── plan-mode.md              ← Plan Mode planning-only agent
├── sub-sections/
│   ├── parallel-tool-use.md      ← Parallel tool calling instructions
│   ├── semantic-search.md        ← Semantic search guidelines
│   ├── replace-string.md         ← File editing instructions
│   ├── apply-patch.md            ← apply_patch tool + patch format (GPT-5.x)
│   ├── todo-list.md              ← Todo list / task tracking instructions
│   ├── terminal.md               ← Terminal command instructions
│   ├── final-answer.md           ← Output formatting rules
│   ├── file-linkification.md     ← File/symbol linking rules (Xs + e$e variants)
│   ├── code-blocks.md            ← Code block formatting rules
│   ├── code-search.md            ← Codebase search + tool-use instructions
│   ├── agent-persistence.md      ← GPT-only "keep going" reminders
│   ├── security-requirements.md  ← OWASP / prompt injection rules
│   ├── operational-safety.md     ← Destructive action guardrails
│   ├── implementation-discipline.md ← "Don't over-engineer" rules
│   ├── communication-style.md    ← Brevity / formatting norms
│   ├── context-management.md     ← Compaction awareness instructions
│   ├── tool-search.md            ← Deferred tool search instructions
│   ├── reminder-instructions.md  ← All reminder instruction variants
│   ├── file-restrictions.md      ← Readonly file restrictions (UEe)
│   ├── repo-memory-learning.md   ← Repository fact learning (Fre)
│   └── vscode-tools.md           ← VS Code API / extensions / commands
└── utility-prompts/
    ├── branch-name.md            ← Git branch name generation
    ├── conversation-title.md     ← Chat title generation
    ├── commit-message.md         ← Commit message generation
    ├── code-explanation.md       ← /explain command prompt
    ├── test-generation.md        ← Test framework recommendation
    ├── terminal-command.md       ← @terminal command help
    ├── conversation-compaction.md ← Conversation summary/compaction
    ├── pr-description.md         ← Pull request title + description
    ├── explore-subagent.md       ← Explore subagent (codebase exploration)
    ├── ask-agent.md              ← Ask Agent (read-only Q&A)
    └── edit-mode-agent.md        ← Edit Mode agent (file-restricted editing)
```

## How to Read These

Each file contains the **reconstructed plain-text** version of what the
`vscpp()` JSX-like rendering produces. The original source is minified
JavaScript with `vscpp("br",null)` for newlines and `vscpp(q,{name:"X"},...)` 
for named XML sections. These have been converted back to readable markdown.

**Dynamic sections** (tool-dependent, model-dependent) are marked with
`{{CONDITIONAL: ...}}` annotations.
