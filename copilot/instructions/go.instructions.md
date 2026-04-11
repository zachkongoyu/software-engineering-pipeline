---
applyTo: "**/*.go"
description: Go style, tooling, and elegance rules. Applies to every .go file in every workspace.
---

# Go

These rules sit on top of `copilot-instructions.md`. When they conflict
with the global constitution, the global constitution wins.

Go is already opinionated. These rules are mostly about resisting the
urge to "improve" on those opinions.

## Baseline

- **Latest stable Go.** Modules, workspaces if the project uses them.
- **Tooling.** `gofmt`, `go vet`, `staticcheck`, `golangci-lint` if
  the project uses it. `go test -race` in CI.
- **Package layout.** Flat when you can, `internal/` when you must,
  never `pkg/` unless the project already has one.

## Shape

- **Small interfaces, defined by the consumer.** `io.Reader`, not
  `MyBigService`. If an interface has more than three methods, it's
  probably wrong.
- **Accept interfaces, return structs.**
- **Zero value is useful.** Design types so the zero value is a valid,
  empty state. Avoid constructor functions when a `var x Foo` works.
- **No stringly-typed enums.** Use a named `int` or `string` type
  plus constants; never switch on a raw string literal in logic code.
- **Dependency injection via parameters and struct fields.** No
  global `http.DefaultClient`, no package-level singletons, no
  `init()` doing real work.

## Errors

- **Errors are values.** Return them. Do not log them and return
  `nil`. Do not panic in library code.
- **Wrap with context.** `fmt.Errorf("load config: %w", err)`. Never
  lose the cause.
- **Sentinel and typed errors, not string matching.** Define
  `var ErrNotFound = errors.New("not found")` and check with
  `errors.Is` / `errors.As`.
- **One style per package.** Don't mix `pkg/errors`, `fmt.Errorf`,
  and typed errors arbitrarily. Pick one.
- **No naked panics.** Panic is for invariant violations the program
  cannot recover from. Not for user input.

## Concurrency

- **Share memory by communicating.** Channels for ownership transfer,
  mutexes for small critical sections. Not both.
- **Cancellation is `context.Context`.** Every function that does IO
  takes a `ctx context.Context` as its first parameter.
- **Goroutines you start, you also stop.** Every `go f()` has a
  defined lifetime — a `ctx`, a `WaitGroup`, or a channel close.
- **No goroutine leaks.** Test with `goleak` if the project uses it.

## Tests

- **Table-driven tests.** One test function per behavior, a slice
  of cases, `t.Run(tc.name, ...)` per case.
- **Subtests name behavior.** `rejects_empty_host`, not `case_1`.
- **Fakes over mocks.** Interfaces + hand-written fake structs beats
  any mock library.
- **`-race` must pass.** Always.
- **No sleep.** If you need time in a test, inject a clock.

## Patterns I like

```go
type UserRepo interface {
    Get(ctx context.Context, id UserID) (User, error)
}

type userRepoPG struct {
    db *sql.DB
}

func (r userRepoPG) Get(ctx context.Context, id UserID) (User, error) {
    const q = `SELECT id, email FROM users WHERE id = $1`
    var u User
    if err := r.db.QueryRowContext(ctx, q, id).Scan(&u.ID, &u.Email); err != nil {
        if errors.Is(err, sql.ErrNoRows) {
            return User{}, ErrNotFound
        }
        return User{}, fmt.Errorf("get user %s: %w", id, err)
    }
    return u, nil
}
```

```go
func TestParseConfig(t *testing.T) {
    cases := []struct {
        name    string
        input   string
        want    Config
        wantErr bool
    }{
        {"rejects empty", ``, Config{}, true},
        {"parses minimal", `{"host":"a","port":1}`, Config{Host: "a", Port: 1}, false},
    }
    for _, tc := range cases {
        t.Run(tc.name, func(t *testing.T) {
            got, err := ParseConfig(tc.input)
            if (err != nil) != tc.wantErr {
                t.Fatalf("err = %v, wantErr %v", err, tc.wantErr)
            }
            if got != tc.want {
                t.Errorf("got %+v, want %+v", got, tc.want)
            }
        })
    }
}
```

## Patterns I reject

- `interface{}` / `any` as a function parameter when a concrete type
  works.
- `if err != nil { return err }` — always wrap with context.
- `log.Fatal` in library code.
- `init()` functions that touch the network, filesystem, or env.
- "Util" packages. Name things by what they do.
