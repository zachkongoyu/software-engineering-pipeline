---
applyTo: "**/*.{ts,tsx,js,jsx,mjs,cjs}"
description: TypeScript / JavaScript style, tooling, and elegance rules. Applies to every TS/JS file in every workspace.
---

# TypeScript / JavaScript

These rules sit on top of `copilot-instructions.md`. When they conflict
with the global constitution, the global constitution wins.

## Baseline

- **TypeScript, strict.** `"strict": true`, `"noUncheckedIndexedAccess":
  true`, `"exactOptionalPropertyTypes": true`, `"noImplicitOverride":
  true`.
- **Tooling.** `eslint` + `@typescript-eslint`, `prettier`, `vitest`
  or `jest`. `tsx` or `node --experimental-strip-types` for runtime.
- **Module system.** ESM. `import`/`export`, never `require`.
- **Package manager.** Whatever the project uses. Don't switch it.

## Shape

- **No `any`.** Ever. Use `unknown` and narrow. If you think you need
  `any`, you need a type guard.
- **Types as documentation.** Prefer explicit return types on exported
  functions. Local inference is fine.
- **Discriminated unions over boolean flags.**
  ```ts
  type Result<T, E> =
    | { kind: "ok"; value: T }
    | { kind: "err"; error: E };
  ```
- **Readonly by default.** `readonly` on fields, `ReadonlyArray<T>`
  on parameters, `as const` on literal configs.
- **`interface` for public contracts, `type` for everything else.**
  Use the one that gives the better error message.
- **Branded types for IDs.**
  ```ts
  type UserId = string & { readonly __brand: "UserId" };
  ```
- **No default exports** in libraries. Named exports only. Defaults
  are fine for React components.

## Errors

- **Throw `Error` subclasses with names.**
  ```ts
  class ConfigError extends Error {
    override name = "ConfigError";
  }
  ```
- **Never swallow.** `try { ... } catch { /* ignore */ }` is a
  bug report.
- **Async errors are still errors.** Await everything or explicitly
  `.catch`. No floating promises.
- **`Result<T, E>`** at boundaries where errors are expected (form
  validation, API responses). Throw for programmer errors.

## React (when present)

- **Function components, hooks only.** No class components.
- **One responsibility per component.** If the name has "And" or
  "With" and a list, split it.
- **Props explicit, no `...rest` forwarding** unless the component
  is a thin wrapper.
- **No `useEffect` for derivation.** Derive during render. `useEffect`
  is for synchronizing with systems outside React (network, DOM, time).
- **Keys are stable IDs, never index.**

## Tests

- **`vitest` / `jest`.** Arrange / Act / Assert. One behavior per test.
- **Names describe behavior**, not the function under test.
- **Fakes over mock libraries.** A hand-written in-memory client is
  clearer than `vi.mock`.
- **No snapshot tests** except for stable, human-reviewed output.

## Patterns I like

```ts
export async function fetchUser(
  id: UserId,
  http: HttpClient,
): Promise<Result<User, UserError>> {
  const res = await http.get(`/users/${id}`);
  if (res.status === 404) return { kind: "err", error: "not-found" };
  if (!res.ok)            return { kind: "err", error: "upstream" };
  return { kind: "ok", value: parseUser(res.body) };
}
```

```ts
const ROLE = ["admin", "member", "guest"] as const;
type Role = typeof ROLE[number];
```

## Patterns I reject

- `any`, `as any`, `@ts-ignore`, `@ts-expect-error` without a tracker
  link.
- `function doThing(opts: { a?: string; b?: number; c?: boolean; d?:
  string[] })` — model the valid combinations with a discriminated
  union.
- `useEffect(() => setX(derive(y)), [y])` — just `const x = derive(y)`.
- Throwing strings or plain objects.
- Default-exporting utility modules.
