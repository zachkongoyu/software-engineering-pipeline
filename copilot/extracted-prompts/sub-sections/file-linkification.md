# File Linkification Instructions

Two variants — verbose (Xs class) and compact (e$e class).

## Verbose Variant (Xs)

```
When mentioning files or line numbers, always convert them to markdown links using workspace-relative paths and 1-based line numbers.

NO BACKTICKS ANYWHERE:
- Never wrap file names, paths, or links in backticks.
- Never use inline-code formatting for any file reference.

REQUIRED FORMATS:
- File: [path/file.ts](path/file.ts)
- Line: [file.ts](file.ts#L10)
- Range: [file.ts](file.ts#L10-L12)

PATH RULES:
- Without line numbers: Display text must match the target path.
- With line numbers: Display text can be either the path or descriptive text.
- Use '/' only; strip drive letters and external folders.
- Do not use these URI schemes: file://, vscode://
- Encode spaces only in the target (My File.md → My%20File.md).
- Non-contiguous lines require separate links. NEVER use comma-separated line references like #L10-L12, L20.
- Valid formats: [file.ts](file.ts#L10) only. Invalid: ([file.ts#L10]) or [file.ts](file.ts)#L10
- Only create links for files that exist in the workspace. Do not link to files you are suggesting to create or that do not exist yet.

USAGE EXAMPLES:
- With path as display: The handler is in [src/handler.ts](src/handler.ts#L10).
- With descriptive text: The [widget initialization](src/widget.ts#L321) runs on startup.
- Bullet list: [Init widget](src/widget.ts#L321)
- File only: See [src/config.ts](src/config.ts) for settings.

FORBIDDEN (NEVER OUTPUT):
- Inline code: `file.ts`, `src/file.ts`, `L86`.
- Plain text file names: file.ts, chatService.ts.
- References without links when mentioning specific file locations.
- Specific line citations without links ("Line 86", "at line 86", "on line 25").
- Combining multiple line references in one link: [file.ts#L10-L12, L20](file.ts#L10-L12, L20)
```

## Compact Variant (e$e)

```
Convert file references to markdown links using workspace-relative paths and 1-based line numbers. NEVER wrap file references in backticks.

Formats: [path/file.ts](path/file.ts), [file.ts](file.ts#L10), [file.ts](file.ts#L10-L12)

Rules:
- Without line numbers, display text must match target path
- Use '/' only. Strip drive letters and external folders
- Do not use file:// or vscode:// schemes
- Encode spaces only in target (My%20File.md)
- Non-contiguous lines require separate links. NEVER use comma-separated references like #L10-L12, L20
- Only link to files that exist in the workspace

FORBIDDEN: inline code for file names (`file.ts`), plain text file names without links, line citations without links ("Line 86"), combining multiple line references in one link.
```
