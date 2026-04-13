# Semantic Search Instructions

Two variants — verbose (Claude 4.5 / GPT) and compact (Claude Sonnet 4 / generic).

## Verbose Variant

```
`semantic_search` is a tool that will find code by meaning, instead of exact text.

Use `semantic_search` when you need to:
- Find code related to a concept but don't know exact naming conventions
- The user asks a question about the codebase and you need to gather context
- Explore unfamiliar codebases
- Understand "what" / "where" / "how" questions about the codebase or the task at hand
- Prefer semantic search over guessing file paths or grepping for terms you're unsure about

Do not use `semantic_search` when:
- You are reading files with known file paths (use `read_file`)
- You are looking for exact text matches, symbols, or functions (use `grep_search`)
- You are looking for specific files (use `file_search`)

Keep each semantic search query to a single concept — `semantic_search` performs poorly when asked about multiple things at once. Break multi-concept questions into separate parallel queries (up to 5 at a time).

EXAMPLES:
GOOD - Specific, focused question with enough context:
- "How does the checkout flow handle failed payment retries?"
- "Where is user input sanitized before it reaches the database?"
- "file upload size validation"
- "how websocket connections are authenticated"

BAD - Vague or keyword-only queries (use grep_search for these):
- "checkout" — no context or intent; too broad
- "upload validation error" — phrase-style, not a question; performs poorly
- "UserService, OrderRepository, CartController" — use grep_search for known symbol names

BAD - Multiple concepts in a single query:
- "How does the checkout flow work, what happens when payment fails, and how are errors shown to the user?" — split into three parallel queries

GOOD - Sequential: use semantic search first, then read specific files:
- Semantic search "How does the job queue handle retries after failure?" → review results → read specific queue implementation file
```

## Compact Variant

Inline rules within `toolUseInstructions` section:
```
For semantic search across the workspace, use semantic_search. For exact text matches, use grep_search. For files by name or path pattern, use file_search. Do not skip search and go directly to read_file unless you are confident about the exact file path.
```

And:
```
If semantic_search returns the full workspace contents, you have all the context.
```
