---
applyTo: "**/*.rs"
description: Rust style, tooling, and elegance rules. Applies to every .rs file in every workspace.
---

# Rust

These rules sit on top of `copilot-instructions.md`. When they conflict
with the global constitution, the global constitution wins.

Rust already enforces a great deal. These rules are mostly about using
the type system aggressively to make illegal states unrepresentable,
writing idiomatic error handling, and keeping unsafe at arm's length.

## Baseline

- **Stable Rust**, latest edition (currently 2024). Use `rustup` to keep
  the toolchain current.
- **Tooling.** `cargo fmt`, `cargo clippy -- -D warnings`,
  `cargo test`, `cargo audit`. No warnings allowed in CI.
- **Workspace.** Multi-crate projects use Cargo workspaces. Shared
  types live in a `domain` or `core` crate; IO in a sibling crate.

## Shape

- **Newtype wrappers for domain IDs.**
  ```rust
  #[derive(Debug, Clone, PartialEq, Eq, Hash)]
  pub struct UserId(String);
  impl UserId {
      pub fn new(s: impl Into<String>) -> Result<Self, DomainError> { ... }
  }
  ```
- **Exhaustive enums for finite states.** If you're matching on a string
  or integer constant, make it an enum. The compiler enforces exhaustiveness.
- **Builder pattern for complex construction**, not fifteen-argument
  constructors or optional fields defaulting to `None`.
- **`#[non_exhaustive]`** on public enums and structs that may grow.
- **Prefer `impl Trait` for return types** when the concrete type is an
  implementation detail.
- **Small traits.** A trait with one method is a good trait. Trait objects
  (`dyn Trait`) are fine at boundaries; generics are fine inside.

## Errors

- **`thiserror` for library errors.** Each error variant is named and
  carries context. Never use `String` as an error type.
  ```rust
  #[derive(Debug, thiserror::Error)]
  pub enum ConfigError {
      #[error("missing required field: {field}")]
      MissingField { field: &'static str },
      #[error("invalid port {port}: must be 1–65535")]
      InvalidPort { port: i64 },
  }
  ```
- **`anyhow` for application code.** `anyhow::Result<T>` with `.context()`
  chains for human-readable error messages at the top of the call stack.
- **`?` everywhere.** Never `.unwrap()` or `.expect()` in library code.
  `expect` is acceptable in tests and `main` only where the panic message
  is a programmer error that will never occur in production.
- **No `panic!` in library crates** except invariant violations that are
  genuinely impossible to recover from. Document them.
- **`Result`, not `Option`, when absence is an error.** `Option<T>` says
  "this might not be there"; `Result<T, E>` says "here's why it isn't".

## Ownership and borrowing

- **Clone only when you mean it.** If you're cloning to satisfy the borrow
  checker, rethink the design — you probably have a lifetime or ownership
  issue worth fixing.
- **`Arc<Mutex<T>>` is a last resort.** Prefer message passing (`tokio::sync::mpsc`,
  channels) for shared mutable state.
- **`Cow<'a, str>` for functions** that sometimes need to own their string
  and sometimes can borrow it.
- **Lifetime annotations are documentation.** Name them meaningfully:
  `'conn`, `'req`, `'static`. Avoid `'a`, `'b` except for trivial cases.

## Async

- **`tokio` as the runtime.** `async fn` all the way; no blocking calls
  on the async thread (`std::thread::sleep`, `std::fs::read` — use
  `tokio::time::sleep`, `tokio::fs::read`).
- **Structured concurrency.** `tokio::spawn` + `JoinSet` or `select!` with
  explicit cancellation. Never leave a spawned task with no way to know it
  ended.
- **`tokio::sync::oneshot` / `mpsc`** for inter-task communication.

## Safety

- **`unsafe` blocks need a `SAFETY:` comment** explaining the invariant
  the programmer is upholding. No exceptions.
- **No `transmute` without a compelling reason and a safety proof.**
- **Fuzz critical parsers.** Use `cargo-fuzz` or `afl` for anything that
  processes untrusted bytes.
- **`derive(Debug)` on secrets is a data leak.** Implement `Debug` manually
  to redact sensitive fields.
  ```rust
  impl fmt::Debug for ApiKey {
      fn fmt(&self, f: &mut fmt::Formatter<'_>) -> fmt::Result {
          f.write_str("ApiKey([redacted])")
      }
  }
  ```

## Tests

- **Unit tests colocated.** `#[cfg(test)] mod tests { ... }` in the same
  file as the code under test.
- **Integration tests in `tests/`.** One file per behaviour area.
- **`proptest` or `quickcheck`** for parsers, serializers, and anything
  with invariants.
- **No `unwrap()` in tests.** Use `?` with `-> Result<(), Box<dyn Error>>`
  or `anyhow::Result<()>` return type so failures produce a real error
  message.
- **Test names describe behaviour.** `#[test] fn rejects_port_zero()`
  not `test_config_1`.

## Patterns I like

```rust
// Total parsing at the boundary; the interior trusts the type
pub fn parse_config(raw: &str) -> Result<Config, ConfigError> {
    let v: serde_json::Value = serde_json::from_str(raw)
        .map_err(|e| ConfigError::Malformed { source: e })?;
    let host = v["host"].as_str()
        .ok_or(ConfigError::MissingField { field: "host" })?
        .to_owned();
    let port = v["port"].as_u64()
        .and_then(|p| u16::try_from(p).ok())
        .ok_or(ConfigError::MissingField { field: "port" })?;
    Ok(Config { host, port })
}
```

```rust
// Exhaustive state machine — new variant = compiler error at every match
#[derive(Debug, Clone, PartialEq)]
pub enum OrderStatus { Pending, Processing, Shipped, Delivered, Cancelled }
```

## Patterns I reject

- `unwrap()` in library code.
- `String` as an error type (`Err("something went wrong".to_string())`).
- `unsafe` without a `SAFETY:` comment.
- `#[allow(clippy::...)]` as a first resort.
- `Arc<Mutex<HashMap<...>>>` when a single-owner design would work.
- Blocking IO on an async thread.
