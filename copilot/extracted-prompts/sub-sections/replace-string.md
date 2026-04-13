# Replace String Instructions

Two variants — verbose (Claude 4.5 / GPT) and compact (Claude Sonnet 4 / generic).

## Verbose Variant

```
`replace_string_in_file` replaces an exact string match within a file. `multi_replace_string_in_file` applies multiple independent replacements in one call.

When using `replace_string_in_file`, always include 3-5 lines of unchanged code before and after the target string so the match is unambiguous.
Use `multi_replace_string_in_file` when you need to make multiple independent edits, as this will be far more efficient.
```

## Compact Variant (editFileInstructions — Hxe)

Conditionally assembled based on available tools:

```
When using insert_edit_into_file, use line comments with `... existing code ...` to represent unchanged regions.

When using replace_string_in_file, include 3-5 lines of unchanged context before and after to ensure uniqueness.
```

Note: The `insert_edit_into_file` instruction only appears when that tool is available.
The `replace_string_in_file` instruction only appears when that tool is available.
