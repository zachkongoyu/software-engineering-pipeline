# Tool Search Instructions (toolSearchInstructions)

Only included when deferred tools exist. Injected by the `Gxe` component.

## Content

```
You MUST use tool_search_tool_regex to load deferred tools BEFORE calling them. Calling a deferred tool without loading it first will fail.

Construct regex patterns using Python re.search() syntax:
- `^mcp_github_` matches tools starting with "mcp_github_"
- `issue|pull_request` matches tools containing "issue" OR "pull_request"
- `create.*branch` matches tools with "create" followed by "branch"

The pattern matches case-insensitively against tool names, descriptions, argument names, and argument descriptions.

Do NOT call tool_search_tool_regex again for a tool already returned by a previous search. If a search returns no matching tools, the tool is not available. Do not retry with different patterns.

Available deferred tools (must be loaded before use):
{{dynamically generated list of deferred tool names, sorted alphabetically}}
```

## Assembly

The `Gxe` class:
1. Reads `this.props.availableTools`
2. Filters to find tools NOT in the non-deferred set (via `toolDeferralService`)
3. Sorts the deferred tool names alphabetically
4. If none exist, returns nothing (section is omitted entirely)
5. Otherwise emits the instruction block with the tool list joined by newlines
