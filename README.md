# Software Engineering Pipeline — User-Level Config

An opinionated, user-level (global) configuration for an end-to-end software
engineering pipeline in VS Code. Targets **GitHub Copilot** and **Claude Code**,
tuned for elegant code in Python, TypeScript / JavaScript, Go, and Rust.

Install once. Every project you open inherits the same craftsmanship bar.

---

## Layout

```
.
├── README.md                         This file
├── INSTALL.md                        Step-by-step install for both tools
│
├── claude/                           Claude Code user-level (~/.claude)
│   ├── CLAUDE.md                     Global memory: principles + workflow
│   ├── settings.json                 Allowed commands, model defaults
│   ├── commands/                     Slash commands (/review, /tdd, ...)
│   └── skills/                       Invocable skills
│       ├── elegant-code/SKILL.md
│       ├── tdd/SKILL.md
│       └── code-review/SKILL.md
│
└── copilot/                          GitHub Copilot user-level (~/.config/copilot)
    ├── copilot-instructions.md       Constitution — always loaded
    ├── vscode-settings.jsonc         Merge into VS Code user settings.json
    │
    ├── agents/                       Orchestrator + sub-agent pipeline
    │   ├── orchestrator.agent.md     Entry point — classifies + dispatches
    │   ├── architect.agent.md        System design, contracts, ADRs
    │   ├── planner.agent.md          Ordered task list from a shaped problem
    │   ├── implementer.agent.md      Minimal diff, one plan item at a time
    │   ├── tester.agent.md           Test strategy + authoring + runs suite
    │   ├── reviewer.agent.md         Findings by severity, no rewrites
    │   ├── security.agent.md         OWASP, supply chain, secrets scan
    │   ├── debugger.agent.md         Reproduce → root cause → fix → pin
    │   ├── refactorer.agent.md       Behavior-preserving improvement
    │   └── documenter.agent.md       READMEs, ADRs, runbooks, API docs
    │
    ├── instructions/                 Auto-attached language rules
    │   ├── python.instructions.md    applyTo: **/*.py
    │   ├── typescript.instructions.md applyTo: **/*.{ts,tsx,js,jsx,...}
    │   ├── go.instructions.md        applyTo: **/*.go
    │   ├── rust.instructions.md      applyTo: **/*.rs
    │   └── tests.instructions.md     applyTo: test files (all languages)
    │
    └── prompts/                      Slash-command verbs in Copilot Chat
        ├── review.prompt.md          /review — correctness + security
        ├── tdd.prompt.md             /tdd — Red → Green → Refactor
        ├── debug.prompt.md           /debug — reproduce, diagnose, fix
        └── refactor.prompt.md        /refactor — behavior-preserving cleanup
```

---

## The agent pipeline

Every non-trivial task starts with the **orchestrator**. It classifies the
request, decides which sub-agents to invoke and in what order, carries state
between stages, and delivers a concise summary at the end.

```
request ──▶ orchestrator
                │
    ┌───────────┼────────────────────┐
    ▼           ▼                    ▼
architect    planner            (small change)
    │           │                    │
    └─────┬─────┘                    │
          ▼                          ▼
      implementer ◀───────────── implementer
          │                          │
          ▼                          ▼
        tester                    tester
          │                          │
          ▼                          ▼
        reviewer  ──▶ security    reviewer
          │                          │
     (if red)                   (if red)
          ▼                          ▼
      debugger/refactorer       debugger
          │                          │
          └─────────┬────────────────┘
                    ▼
               documenter (if contracts/ops changed)
                    │
                    ▼
                 summary
```

To switch into a mode in VS Code, click the mode picker in Copilot Chat
and select the agent by name (e.g. **orchestrator**, **tester**, **reviewer**).

---

## Philosophy

These files encode one commitment: **elegance is the default**.

1. Clarity over cleverness.
2. Delete before you write — the best code is the code you didn't add.
3. Make illegal states unrepresentable; let the types do the talking.
4. Small, pure functions at the core; side effects at the edges.
5. Name things so the code reads like prose.
6. Fail loudly, close to the cause.
7. Tests describe behavior; they are the spec, not the scaffolding.
8. Abstract only on the third repetition, never on the first.

---

## Install

See `INSTALL.md`. For the impatient, a one-shot script is at the bottom of
that file.
