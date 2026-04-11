---
name: elegant-code
description: The canonical rules for writing elegant code — clarity over cleverness, small surface area, functional core / imperative shell, illegal states unrepresentable, and the disciplines that keep a codebase honest. Invoke this skill whenever you are writing, refactoring, or reviewing code and the user has not overridden the house style.
---

# Elegant Code

Elegance is not decoration. It is the shortest honest path from intent to
behavior, written so the next reader understands it without help.

## The ten principles

1. **Clarity over cleverness.** If a line needs a comment to explain its
   trick, rewrite the line instead. A comment is a bug report on the code.
2. **Delete before you write.** Every line of code is a liability. Before
   adding one, ask whether deleting another would solve the problem.
3. **Small, orthogonal pieces.** A function should do one thing at one level
   of abstraction. A module should have one reason to change.
4. **Functional core, imperative shell.** Push IO, time, randomness, and
   mutation to the edges. The core should be a pure function of its inputs.
5. **Make illegal states unrepresentable.** Model with types, not runtime
   checks. `Option`/`Result`, sum types, newtypes, and exhaustiveness are
   free correctness.
6. **Explicit over implicit.** No hidden globals. No "smart" auto-wiring.
   Dependencies flow in through parameters or constructors.
7. **Names carry intent.** `user_ids_active_last_7d` beats `ids` every time.
   A good name makes a comment redundant.
8. **Fail loudly, close to the cause.** Validate at the boundary. Raise a
   specific error. Do not paper over failure with defaults.
9. **Rule of three.** Abstract only on the third occurrence. Two points
   make a line through infinitely many curves; three start to constrain it.
10. **YAGNI.** Do not add configuration, hooks, or extension points that the
    current change does not need.

## Heuristics you can apply in real time

- **If a function is longer than one screen**, it is doing more than one
  thing. Split it by level of abstraction, not by line count.
- **If a parameter list is longer than three or four**, either the function
  is doing too much or the parameters want to be a struct.
- **If a boolean flag changes branches inside a function**, split it into
  two functions and let the caller choose.
- **If the same three lines appear twice**, leave them. If they appear a
  third time in a *recognizably identical* shape, extract.
- **If you are writing a comment explaining "why we do this weird thing"**,
  promote the weird thing into a named helper whose name answers the
  question.
- **If you are writing a try/except that catches `Exception`**, stop. Catch
  the specific class, or let it propagate.
- **If you find yourself adding a flag to a class to remember whether it was
  initialized**, the type is wrong. Use a constructor that cannot produce an
  uninitialized value.

## Before / after in four flavors

### Python

```python
# Before
def get_user(id, db, cache=None, logger=None, retries=3):
    if not id:
        return None
    try:
        if cache:
            u = cache.get(id)
            if u:
                return u
        for _ in range(retries):
            try:
                u = db.fetch("users", id)
                if u:
                    if cache:
                        cache.set(id, u)
                    return u
            except Exception as e:
                if logger:
                    logger.warn(str(e))
        return None
    except Exception:
        return None
```

```python
# After
def get_user(user_id: UserId, users: UserRepo) -> User:
    """Return the user or raise UserNotFound."""
    return users.get(user_id)
```

The retries, caching, and logging are not the domain logic — they are
concerns of the `UserRepo` implementation, injected at the edge. The domain
function is a pure lookup with a total contract (`User` or `UserNotFound`).

### TypeScript

```ts
// Before
function parseConfig(s: any): any {
  const o = JSON.parse(s);
  if (o && o.port) o.port = Number(o.port);
  return o;
}
```

```ts
// After
type Config = { readonly host: string; readonly port: number };

function parseConfig(raw: string): Config {
  const parsed = JSON.parse(raw) as unknown;
  if (!isRecord(parsed)) throw new Error("config: not an object");
  const host = asString(parsed.host, "host");
  const port = asInt(parsed.port, "port");
  return { host, port };
}
```

### Go

```go
// Before
func Handle(w http.ResponseWriter, r *http.Request) {
    b, _ := io.ReadAll(r.Body)
    var m map[string]interface{}
    json.Unmarshal(b, &m)
    name := m["name"].(string)
    fmt.Fprintf(w, "hello %s", name)
}
```

```go
// After
type greetRequest struct {
    Name string `json:"name"`
}

func handleGreet(w http.ResponseWriter, r *http.Request) {
    var req greetRequest
    if err := json.NewDecoder(r.Body).Decode(&req); err != nil {
        http.Error(w, "invalid body", http.StatusBadRequest)
        return
    }
    if req.Name == "" {
        http.Error(w, "name required", http.StatusBadRequest)
        return
    }
    fmt.Fprintf(w, "hello %s", req.Name)
}
```

### Rust

```rust
// Before
fn discount(price: f64, kind: &str) -> f64 {
    if kind == "member" { price * 0.9 }
    else if kind == "staff" { price * 0.7 }
    else { price }
}
```

```rust
// After
enum Customer { Guest, Member, Staff }

fn discount(price: Money, customer: Customer) -> Money {
    match customer {
        Customer::Guest  => price,
        Customer::Member => price * 0.9,
        Customer::Staff  => price * 0.7,
    }
}
```

The `match` is exhaustive — adding a new `Customer` variant is a compile
error everywhere it matters. The stringly-typed `kind` is gone forever.

## When the user asks for a change

1. Read the surrounding code first. Never edit what you haven't read.
2. State the smallest change that satisfies the request.
3. If the change would violate these principles, note the tension and pick
   the least-bad path (usually: ship the minimal fix now, file the cleanup).
4. Keep refactors and behavior changes in separate diffs.
5. Run the project's lint / type / test commands. Report results honestly.
