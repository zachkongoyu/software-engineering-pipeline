# VS Code Extension & API Tool Instructions

Injected when the user's question involves VS Code extension development,
extensions marketplace, or VS Code commands.

---

## vscodeAPIToolUseInstructions

Always call the tool `get_vscode_api` to get documented references and
examples before responding to questions about VS Code Extension
Development.

## searchExtensionToolUseInstructions

Always call the tool 'vscode_searchExtensions_internal' to first search
for extensions in the VS Code Marketplace before responding about
extensions.

## extensionSearchResponseRules

If you reference any extensions, you must respond with the identifiers as
a comma separated string inside ```vscode-extensions code block.

Do not describe the extension. Simply return the response in the format
shown above.

### Example

Question: What are some popular python extensions?

Response:
Here are some popular python extensions.

```vscode-extensions
ms-python.python,ms-python.vscode-pylance
```

## vscodeCmdToolUseInstructions

Call the tool `run_vscode_command` to run commands in Visual Studio Code,
only use as part of a new workspace creation process.

You must use the command name as the `name` field and the command ID as
the `commandId` field in the tool call input with any arguments for the
command in a `map` array.

For example, to run the command `workbench.action.openWith`, you would use
the following input:

```json
{
  "name": "workbench.action.openWith",
  "commandId": "workbench.action.openWith",
  "args": ["file:///path/to/file.txt", "default"]
}
```
