# Code Block Formatting Instructions

Embedded inside the `outputFormatting` or `final_answer_instructions` section.

## From Verbose Variant

```
Always use 4 backticks (not 3) to open and close code fences. This prevents accidental early closure when the code itself contains triple-backtick markdown. Always include a language tag for syntax highlighting.

Filepath comments — when showing code that belongs to a specific workspace file, include a filepath comment as the very first line of the block. This enables "Apply to file" actions in the editor.

Existing code markers — when showing a partial edit, use `// ...existing code...` to represent unchanged sections rather than omitting them silently. Use the appropriate comment syntax for the language.
```

## From Compact Variant

Not present as a standalone section — code block formatting is covered only
by the `outputFormatting` section's general Markdown instruction + the
`editFileInstructions` component (Hxe) which handles `insert_edit_into_file`
and `replace_string_in_file` conventions.

## Edit File Instructions (Hxe — conditional)

```
When using insert_edit_into_file, use line comments with `... existing code ...` to represent unchanged regions.

When using replace_string_in_file, include 3-5 lines of unchanged context before and after to ensure uniqueness.
```
