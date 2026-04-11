# Global Engineering Memory

This file is loaded on every Claude Code session as user-level memory. It
describes how I want Claude to think, write code, and collaborate with me
across every project on this machine.

## Who I am

I am a software engineer working primarily in **Python**, **TypeScript /
JavaScript**, **Go**, and **Rust**, from VS Code. I care about craft. I would
rather ship less code that is clear than more code that is clever.

## Prime directive — elegance is the default

Every change you propose must optimize for the reader who will see this code
six months from now with no context. That reader is usually me.

Concretely:

- **Clarity over cleverness.** If a one-liner needs a comment to be
  understood, expand it. If a helper name needs a comment, rename it.
- **Delete before you write.** The best diff is a negative one. Before adding
  code, check whether removing code solves the problem.
- **Small surface area.** Prefer a handful of small, orthogonal functions
  over one large function with flags.
- **Functional core, imperative shell.** Push side effects (IO, time,
  randomness, network) to the edges. The core should be pure and trivially
  testable.
- **Make illegal states unrepresentable.** Use the type system to rule out
  invalid combinations at compile time rather than guarding them at runtime.
- **Explicit over implicit.** No hidden globals, no magic, no metaclass
  gymnastics, no reflection when a function will do.
- **Fail loudly, fail close to the cause.** Validate at boundaries. Raise
  specific errors. Never silently swallow exceptions.
- **Names are documentation.** A good name erases the need for a comment.
- **Rule of three.** Don't abstract on the first occurrence or the second.
  Wait until the pattern is stable.
- **YAGNI.** Do not add configuration, hooks, or extension points I have not
  asked for.

## Workflow norms

1. **Understand before you touch.** Read the relevant files first. Never edit
   code you haven't read. If the context is ambiguous, ask one clarifying
   question rather than guessing.
2. **Plan, then act.** For any change bigger than a few lines, produce a
   short plan (bullet list of files and what will change) before editing.
3. **Small, reviewable diffs.** Group related edits into a single commit.
   Never mix refactors with behavioral changes.
4. **Tests go with the change.** A bug fix starts with a failing test. A
   feature ships with tests that pin down its behavior.
5. **Run the suite.** After editing, run the relevant lint / type-check /
   test commands. Report the results honestly — "ran, all green" or "ran, 3
   failures, here they are."
6. **Commit message hygiene.** Imperative mood. Subject ≤ 72 chars. Body
   explains *why*, not *what*. No Claude signatures unless I ask.
7. **Do not touch unrelated files.** If you notice other problems, mention
   them at the end; do not fix them in the same change.

## Communication style

- Be direct and technical. Skip throat-clearing and praise.
- Answer the question first, then add context if needed.
- When you are uncertain, say so and estimate your confidence.
- When you make a mistake, own it in one line and fix it.
- Prose responses: paragraphs, not bullet soup. Bullets only when the list is
  genuinely enumerable.
- Code responses: show the diff or the whole small function, not both.

## Language defaults

Detailed rules live in the skills and per-language files, but the defaults
are:

- **Python** — 3.12+, type hints everywhere, `ruff` + `mypy --strict`,
  `pytest`, `pydantic` for boundaries, `dataclasses` or `pydantic` models
  over dicts.
- **TypeScript** — `strict: true`, no `any`, `eslint` + `prettier`, `vitest`
  or `jest`. Prefer `unknown` + narrowing over type assertions.
- **Go** — idiomatic Go, `gofmt`, `go vet`, `staticcheck`, small interfaces,
  errors wrapped with `%w`, table-driven tests.
- **Rust** — `cargo fmt`, `cargo clippy -- -D warnings`, `Result<T, E>` over
  panics, newtype wrappers to make illegal states unrepresentable.

## What I never want

- `TODO` / `FIXME` added without a reason and a link to track it.
- `except Exception: pass` and its cousins in any language.
- Defensive code against impossible states (trust the types).
- "Backward compatibility" hacks for code that has never shipped.
- Comments that restate the code in English.
- Framework soup — pulling in a dependency to save three lines.
- Generated boilerplate I didn't ask for (CI files, Dockerfiles, READMEs).

## Slash commands available

- `/review` — review the staged diff or the file under cursor against the
  elegance principles.
- `/refactor` — propose a refactor that makes the target clearer and
  smaller, then apply it.
- `/tdd` — drive a change test-first: red, green, refactor.
- `/explain` — walk through a piece of code the way a senior engineer would
  explain it to a peer.

## Skills available

- `elegant-code` — the canonical principles, with examples.
- `tdd` — the red-green-refactor loop.
- `code-review` — a reviewer's checklist, ordered by severity.
