---
applyTo: "**/*.py"
description: Python style, tooling, and elegance rules. Applies to every .py file in every workspace.
---

# Python

These rules sit on top of `copilot-instructions.md`. When they conflict
with the global constitution, the global constitution wins.

## Baseline

- **Python 3.12+.** Use `match`, structural typing, `TypeAlias`,
  `Self`, `override`.
- **Type hints on every function and method, always.** No untyped
  public APIs. `Any` is a smell; use `object` or `unknown`-style
  narrowing.
- **Tooling.** `ruff` (formatter + linter), `mypy --strict`, `pytest`,
  `uv` for dependency management.
- **Imports.** Stdlib, third-party, local — three groups, `ruff`
  sorts them. No wildcard imports. No relative imports past one dot.

## Shape

- **Dataclasses or Pydantic, not dicts.** A dict keyed by strings is a
  class wearing a disguise. Model with `@dataclass(frozen=True,
  slots=True)` by default; Pydantic at trust boundaries (HTTP, DB,
  config).
- **Enums over string literals.** If a value has a fixed set of
  possibilities, make it a `StrEnum` or a `Literal[...]` union.
- **Functions over classes.** A class with a constructor and one
  method is a function. Write the function.
- **Pure core, impure shell.** Put `open`, `requests`, `datetime.now`,
  and `random` at the edge. The core should be a pure function of its
  inputs.
- **No `**kwargs` in public APIs** unless you are literally
  forwarding. It erases the signature.

## Errors

- **Specific exceptions.** Define `class ConfigError(Exception)` and
  raise it. Never `raise Exception("bad")`.
- **Never bare `except`.** Never `except Exception: pass`. Catch the
  smallest class, re-raise with context using `raise ... from e`.
- **Validate at the boundary.** Pydantic or explicit checks at the
  edges; the interior trusts its types.

## Tests

- **`pytest`, not `unittest`.** Fixtures over setUp/tearDown.
- **Test names describe behavior.** `test_parse_config_rejects_empty_host`.
- **Parametrize edge cases.** `@pytest.mark.parametrize`.
- **`hypothesis`** for parsers, serializers, anything with
  round-trips or invariants.
- **No sleeps, no network, no wall-clock** in tests. Inject a clock.

## Patterns I like

```python
from dataclasses import dataclass
from typing import Self

@dataclass(frozen=True, slots=True)
class UserId:
    value: str

    @classmethod
    def parse(cls, raw: str) -> Self:
        if not raw or len(raw) > 64:
            raise ValueError(f"invalid user id: {raw!r}")
        return cls(raw)
```

```python
from enum import StrEnum

class Role(StrEnum):
    ADMIN = "admin"
    MEMBER = "member"
    GUEST = "guest"

def discount(role: Role) -> float:
    match role:
        case Role.ADMIN:  return 0.5
        case Role.MEMBER: return 0.1
        case Role.GUEST:  return 0.0
```

## Patterns I reject

- `def get(id, db=None, cache=None, logger=None, retries=3)` — turn
  this into an injected repository with a total contract.
- `except Exception as e: logger.warn(e); return None` — swallowing
  failure is lying to the caller.
- `if isinstance(x, dict) and "port" in x and isinstance(x["port"], int)`
  — parse once at the boundary with Pydantic and trust the type
  afterwards.
- `from module import *` — always.
