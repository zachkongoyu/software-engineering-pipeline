# Install Guide

All files are **user-level** — install once and every project inherits them.

---

## 1. GitHub Copilot

### 1a. Copy the files

```bash
# macOS / Linux
mkdir -p ~/.config/copilot
cp -R copilot/agents        ~/.config/copilot/agents
cp -R copilot/instructions  ~/.config/copilot/instructions
cp -R copilot/prompts       ~/.config/copilot/prompts
cp    copilot/copilot-instructions.md  ~/.config/copilot/copilot-instructions.md
```

```powershell
# Windows (PowerShell)
New-Item -ItemType Directory -Force -Path $HOME\.config\copilot | Out-Null
Copy-Item copilot\agents       $HOME\.config\copilot\agents       -Recurse -Force
Copy-Item copilot\instructions $HOME\.config\copilot\instructions -Recurse -Force
Copy-Item copilot\prompts      $HOME\.config\copilot\prompts      -Recurse -Force
Copy-Item copilot\copilot-instructions.md $HOME\.config\copilot\copilot-instructions.md
```

### 1b. Wire VS Code

Open your **user** `settings.json` (`Cmd/Ctrl+Shift+P` → *Preferences: Open User Settings (JSON)*) and merge in the contents of `copilot/vscode-settings.jsonc`. The key entries are:

```jsonc
{
  "github.copilot.chat.codeGeneration.useInstructionFiles": true,
  "github.copilot.chat.codeGeneration.instructions": [
    { "file": "${userHome}/.config/copilot/copilot-instructions.md" }
  ],
  "chat.instructionsFilesLocations": {
    "${userHome}/.config/copilot/instructions": true
  },
  "chat.promptFiles": true,
  "chat.promptFilesLocations": {
    "${userHome}/.config/copilot/prompts": true
  },
  "chat.modeFilesLocations": {
    "${userHome}/.config/copilot/agents": true
  }
}
```

> **Windows:** replace `${userHome}` with `${env:USERPROFILE}` throughout.

### 1c. What each piece does

| Path | What it does |
|------|-------------|
| `copilot-instructions.md` | The constitution. Loaded on every Copilot interaction. Sets the elegance bar and pipeline rules. |
| `agents/orchestrator.agent.md` | Entry-point agent. Classifies requests, runs the pipeline, dispatches sub-agents. Switch to it in the Copilot Chat mode picker. |
| `agents/architect.agent.md` | Designs components, contracts, and ADRs. |
| `agents/planner.agent.md` | Breaks a shaped problem into an ordered task list. |
| `agents/implementer.agent.md` | Writes the minimal diff for one plan item. |
| `agents/tester.agent.md` | Test strategy + test authoring + runs the suite. |
| `agents/reviewer.agent.md` | Critiques diffs by severity. |
| `agents/security.agent.md` | OWASP threat model, supply chain, secrets scan. |
| `agents/debugger.agent.md` | Reproduces, diagnoses, fixes, and pins bugs. |
| `agents/refactorer.agent.md` | Behavior-preserving cleanup. |
| `agents/documenter.agent.md` | READMEs, ADRs, runbooks, API docs. |
| `instructions/*.instructions.md` | Auto-attached language rules (matched by `applyTo` glob). |
| `prompts/review.prompt.md` | `/review` — correctness + security in one pass. |
| `prompts/tdd.prompt.md` | `/tdd $BEHAVIOR` — Red → Green → Refactor. |
| `prompts/debug.prompt.md` | `/debug $FAILURE` — reproduce, diagnose, fix. |
| `prompts/refactor.prompt.md` | `/refactor` — behavior-preserving improvement. |

### 1d. Verify

1. Open any `.py`, `.ts`, `.go`, or `.rs` file. In Copilot Chat, ask:
   *"What instructions are active?"* — you should see `copilot-instructions.md`
   and the matching language file named.
2. Open the Copilot Chat mode picker — you should see **orchestrator** plus
   all sub-agents listed.
3. Type `/` in Copilot Chat — you should see `review`, `tdd`, `debug`,
   `refactor` in the prompt list.

---

## 2. Claude Code (user-level)

```bash
# macOS / Linux
mkdir -p ~/.claude
cp    claude/CLAUDE.md     ~/.claude/CLAUDE.md
cp    claude/settings.json ~/.claude/settings.json
cp -R claude/commands      ~/.claude/commands
cp -R claude/skills        ~/.claude/skills
```

```powershell
# Windows
New-Item -ItemType Directory -Force -Path $HOME\.claude | Out-Null
Copy-Item claude\CLAUDE.md     $HOME\.claude\CLAUDE.md
Copy-Item claude\settings.json $HOME\.claude\settings.json
Copy-Item claude\commands      $HOME\.claude\commands -Recurse -Force
Copy-Item claude\skills        $HOME\.claude\skills   -Recurse -Force
```

### Verify

Open any project in Claude Code and run:
- `/memory` — `~/.claude/CLAUDE.md` should appear as loaded.
- `/help` — user commands (`/review`, `/refactor`, `/tdd`, `/explain`) should be listed.

---

## One-shot install script (macOS / Linux)

Save as `install.sh` in this directory and run `bash install.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail

COPILOT_DIR="$HOME/.config/copilot"
CLAUDE_DIR="$HOME/.claude"

echo "→ Installing Copilot config to $COPILOT_DIR"
mkdir -p "$COPILOT_DIR"
cp -R copilot/agents        "$COPILOT_DIR/agents"
cp -R copilot/instructions  "$COPILOT_DIR/instructions"
cp -R copilot/prompts       "$COPILOT_DIR/prompts"
cp    copilot/copilot-instructions.md "$COPILOT_DIR/copilot-instructions.md"

echo "→ Installing Claude Code config to $CLAUDE_DIR"
mkdir -p "$CLAUDE_DIR"
cp    claude/CLAUDE.md     "$CLAUDE_DIR/CLAUDE.md"
cp    claude/settings.json "$CLAUDE_DIR/settings.json"
cp -R claude/commands      "$CLAUDE_DIR/commands"
cp -R claude/skills        "$CLAUDE_DIR/skills"

echo "✓ Done. Merge copilot/vscode-settings.jsonc into your VS Code user settings.json."
```
