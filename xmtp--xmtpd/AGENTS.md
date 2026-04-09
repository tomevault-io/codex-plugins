## General Best Practices
- Avoid variable shadowing.
- Do not over-nest control flow (e.g., nested `if` or `for` blocks).
- Avoid `init()` functions unless absolutely necessary.
- Keep functions small and focused.
- Prefer composition over inheritance (via embedding).
- Use the functional options pattern for constructors where flexibility is needed.

## Interfaces

- Avoid defining interfaces until you need them.
- Do not return interfaces from constructors or public APIs.
- Define interfaces on the consumer side, not the producer side.
- Keep interfaces small and focused (generally 1–2 methods).

## Structs and Methods

- Avoid embedding pointer types unless necessary.
- Don’t overuse getters/setters — prefer public fields when it makes sense.
- Use value receivers when the method doesn't mutate state or require pointer semantics.

## Packages and Imports

- Do not use `util`, `common`, or similarly vague package names.
- Avoid package name collisions by using clear, unique names.
- Do not expose unnecessary symbols (keep exported API minimal).

## Slices

- Distinguish between nil and empty slices.
- Avoid memory leaks from slicing large arrays.
- Always check the capacity when copying or appending slices.
- Preallocate slice capacity when size is known ahead of time.

## Maps

- Always initialize maps before use.
- Check existence with the two-value assignment (`val, ok := m[key]`).
- Be aware that ranging over a map is in random order.

## Error Handling

- Always check errors — don’t ignore them.
- Wrap errors with context when rethrowing.
- Avoid panics except in truly unrecoverable cases.
- Use `errors.Is` and `errors.As` for error comparison in Go 1.20+.

## Goroutines and Concurrency

- Always `defer cancel()` when using `context.WithCancel`.
- Do not leak goroutines — ensure they exit cleanly.
- Avoid data races — use mutexes or channels appropriately.
- Never close a channel from the receiving side.

## Testing and Debugging

- Name tests consistently: `TestXxx`, `BenchmarkXxx`, `ExampleXxx`.
- Use table-driven tests where possible.
- Avoid global state in tests.
- Use `t.Helper()` in helper functions to improve error tracebacks.

## Code Style and Tools

- Keep imports grouped and ordered: stdlib, external, internal.
- Avoid magic numbers — use named constants.
- Prefer explicit over implicit — especially in exported APIs.

## Go 1.18+ Generics

- Only use generics when they simplify code or add real flexibility.
- Avoid over-engineering with type parameters.
- Be cautious with constraint complexity — keep things readable.

## Numbers and Types

- Avoid using default `int` unless the size is appropriate.
- Watch for integer overflows and underflows.
- Avoid precision issues with `float32`/`float64` — prefer `math/big` if needed.

## Miscellaneous

- Prefer `any` to `interface{}`, but avoid either unless necessary
- Do not rely on map or slice comparison — use `reflect.DeepEqual` or a library.
- Avoid relying on goroutine scheduling or map iteration order.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/xmtp)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/xmtp)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
