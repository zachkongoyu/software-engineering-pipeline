# Parallel Tool Use Instructions

Two variants exist — verbose (used by Claude 4.5 / GPT) and compact (used by Claude Sonnet 4 / generic).

## Verbose Variant

```
Calling multiple tools in parallel is highly ENCOURAGED, especially for operations such as reading files, creating files, or editing files. If you think running multiple tools can answer the user's question, prefer calling them in parallel whenever possible.

You are encouraged to call functions in parallel if you think running multiple tools can answer the user's question to maximize efficiency by parallelizing independent operations. This reduces latency and provides faster responses to users.

Cases encouraged to parallelize tool calls when no other tool calls interrupt in the middle:
- Reading multiple files for context gathering instead of sequential reads
- Creating multiple independent files (e.g., source file + test file + config)
- Applying patches to multiple unrelated files

Dependency rules:
- Read-only + independent → parallelize encouraged
- Write operations on different files → safe to parallelize
- Read then write same file → must be sequential
- Any operation depending on prior output → must be sequential

Maximum calls: Up to 15 tool calls can be made in a single parallel invocation.
```

## Compact Variant

```
Call independent tools in parallel, but do not call semantic_search in parallel. Call dependent tools sequentially.
```

## Parallelization Strategy (separate sub-section, compact models)

Two sub-variants:

### aBt (emphatic)
```
You may parallelize independent read-only operations when appropriate. For context gathering, batch the reads you've already decided you need rather than searching speculatively. Get enough context to act, then proceed with implementation.
```

### sBt (standard)
```
You may parallelize independent read-only operations when appropriate.
```
