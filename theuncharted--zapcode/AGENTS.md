# AGENTS.md

> Standard agent instructions for the `zapcode` project.
> Read this before writing any code.

---

## What this project is

**Zapcode** is a minimal, secure TypeScript subset interpreter written in Rust,
designed specifically to execute code written by AI agents. It is the TypeScript equivalent of
[pydantic/monty](https://github.com/pydantic/monty).

The core thesis: LLMs produce faster, cheaper, more reliable results when they write code instead of
making sequential tool calls. Zapcode makes that possible for TypeScript/JavaScript stacks without
containers, sandbox services, or running untrusted code directly on the host.

**What Zapcode can do:**
- Execute a safe subset of TypeScript — enough for an agent to express what it wants to do
- Block all host access by default: filesystem, env vars, network, `require`, `import`
- Expose host functions to the sandbox — only functions you explicitly register
- Snapshot VM state to bytes at external function call boundaries — resume later in any process
- Start in microseconds (~2 µs for simple expressions)
- Be called from Rust, TypeScript/JavaScript (napi-rs), Python (PyO3), or WebAssembly
- Enforce resource limits: memory, execution time, stack depth, allocation count

**What Zapcode cannot do (by design):**
- Access the standard library beyond a safe subset (`console`, `JSON`, `Math`, `Array`, `Object`, `Promise`)
- Use `import` / `require` / dynamic imports
- Access `process`, `globalThis`, `eval`, `Function()`, `setTimeout`/`setInterval`
- Use proxies, WeakMap/WeakRef, `Symbol`, or `with` statements
- Execute regular expressions (parsed but execution is a no-op)
- Use `var` declarations (use `let`/`const`)

---

## Repository layout

```
zapcode/
├── crates/
│   ├── zapcode-core/       # Parser, IR, bytecode compiler, VM, snapshot
│   │   ├── src/
│   │   │   ├── parser/      # oxc_parser integration — AST → ZapcodeIR
│   │   │   │   ├── mod.rs   # AST walker (oxc → IR)
│   │   │   │   └── ir.rs    # IR type definitions
│   │   │   ├── compiler/    # ZapcodeIR → Bytecode
│   │   │   │   ├── mod.rs   # Compiler logic
│   │   │   │   └── instruction.rs  # Instruction enum (~55 opcodes)
│   │   │   ├── vm/          # Stack-based bytecode executor
│   │   │   │   ├── mod.rs   # VM main loop, dispatch, ZapcodeRun entry point
│   │   │   │   └── builtins.rs  # Built-in functions (console, Math, JSON, etc.)
│   │   │   ├── value.rs     # Value enum — runtime type system
│   │   │   ├── snapshot.rs  # Serialize/deserialize mid-execution VM state
│   │   │   ├── sandbox.rs   # Resource limits and tracking
│   │   │   ├── error.rs     # ZapcodeError — all error types
│   │   │   └── lib.rs       # Public API re-exports
│   │   ├── tests/           # 14 test files, 214+ tests
│   │   └── benches/         # divan benchmarks
│   ├── zapcode-js/         # napi-rs bindings → @unchartedfr/zapcode npm package
│   ├── zapcode-py/         # PyO3 bindings → zapcode pip package
│   └── zapcode-wasm/       # wasm-bindgen target for browser/edge use
├── AGENTS.md                # This file
├── CLAUDE.md                # Claude Code-specific guidance (references this file)
├── Cargo.toml               # Workspace root
└── README.md                # User-facing docs with benchmarks
```

---

## Architecture overview

### Pipeline

```
TypeScript source → parser (oxc) → ZapcodeIR → compiler → Bytecode → VM → Result/Snapshot
```

### Parser

Uses **[oxc_parser](https://github.com/oxc-project/oxc)** — the fastest TypeScript parser in Rust.
The parser walks the oxc AST and emits `ZapcodeIR`. Unsupported syntax produces
`ZapcodeError::UnsupportedSyntax` with span information.

### Supported syntax subset

| Feature | Status |
|---|---|
| `const`, `let` declarations | ✅ |
| Functions (declarations, arrows, expressions) | ✅ |
| `async function`, `await` | ✅ |
| Classes (`constructor`, methods, `extends`, `super`, `static`) | ✅ |
| Generators (`function*`, `yield`, `.next()`) | ✅ |
| Control flow (`if`, `for`, `while`, `do-while`, `switch`, `for-of`) | ✅ |
| `try` / `catch` / `finally`, `throw` | ✅ |
| Closures with mutable capture | ✅ |
| Destructuring (object and array) | ✅ |
| Spread/rest operators | ✅ |
| Optional chaining `?.` | ✅ |
| Nullish coalescing `??` | ✅ |
| Template literals | ✅ |
| Type annotations, interfaces, type aliases | Stripped at parse time |
| String methods (30+) | ✅ |
| Array methods (25+) | ✅ |
| Math, JSON, Object, Array, Promise | ✅ |
| `import` / `require` / `eval` | ❌ sandbox violation |
| Regular expressions | Parsed, not executed |
| `var` declarations | ❌ not supported |
| Decorators, Symbol, WeakMap/WeakSet | ❌ not supported |

### Value system

```rust
pub enum Value {
    Undefined,
    Null,
    Bool(bool),
    Int(i64),
    Float(f64),
    String(Arc<str>),
    Array(Vec<Value>),
    Object(IndexMap<Arc<str>, Value>),
    Function(Closure),
    BuiltinMethod { object_name: Arc<str>, method_name: Arc<str> },
    Generator(GeneratorObject),
}
```

Arrays and objects are **value types** (owned `Vec<Value>` / `IndexMap`), not reference types.
Mutation works via `SetIndex`/`SetProperty` instructions that push the modified value back
onto the stack for store-back to the variable.

### VM

Stack-based bytecode VM (~55 instructions). Single-threaded, no Tokio runtime inside the VM.

**Suspension at external calls**: when the VM encounters a `CallExternal` instruction for a
registered external function, it:
1. Captures the VM state into a `ZapcodeSnapshot`
2. Returns `VmState::Suspended { function_name, args, snapshot }`

The caller resolves the external function and calls `snapshot.resume(return_value)`.

### Snapshotting

`ZapcodeSnapshot` uses `postcard` for serialization. Snapshots are small (< 2 KB for typical
agent code). They can be stored in any bytes store (DB, Redis, S3).

```rust
let bytes: Vec<u8> = snapshot.dump()?;
let restored = ZapcodeSnapshot::load(&bytes)?;
let final_state = restored.resume(return_value)?;
```

---

## Sandbox invariants — NEVER violate

1. **No host filesystem access** — `std::fs` is forbidden in `zapcode-core`
2. **No env var access** — `std::env::var` is forbidden in `zapcode-core`
3. **No network access** — `std::net`, `tokio::net` are forbidden in `zapcode-core`
4. **No `eval` equivalent** — no mechanism to compile new code at runtime from within the sandbox
5. **Resource limits are enforced** — memory, time, stack depth, allocation count checked during execution

If you are ever unsure whether something violates sandbox invariants: it does. Block it.

---

## Key implementation patterns

### Builtin dispatch

Methods on arrays, strings, and global objects (Math, JSON, etc.) use a `BuiltinMethod` value
variant. The VM stores the `last_receiver` when a property access returns a function or builtin,
and `last_global_name` for known global objects.

### Closure capture

Closures capture variables via `local_names` stored in `CompiledFunction`. At `CreateClosure` time,
all locals from all active frames are captured by name. Captured variables are injected as globals
when the closure is called.

### Generator state

Generator objects are stored in the globals `HashMap` with `__gen_{id}` keys during execution.
When a generator finishes (done=true), its key is removed to prevent unbounded growth.

### Classes

Classes are represented as objects with special keys:
- `__class_name__` — class name for instanceof
- `__constructor__` — constructor closure
- `__prototype__` — object containing instance methods
- `__super__` — reference to super class (for inheritance)
- Static methods are stored directly on the class object

### Promise

Internal promises are plain objects with `{__promise__: true, status, value/reason}`.
`await` on an external call suspends the VM; `await` on an internal `Promise.resolve()`
is handled entirely inside the VM.

---

## How to add a new language feature

1. **Check the supported subset table above.** Features listed as unsupported are excluded intentionally.
2. **Add parser support** in `crates/zapcode-core/src/parser/`. Emit `ZapcodeError::UnsupportedSyntax` for unhandled nodes.
3. **Add compiler support** in `crates/zapcode-core/src/compiler/`. Prefer reusing existing instructions.
4. **Add VM dispatch** in `crates/zapcode-core/src/vm/mod.rs`. Use `push_call_frame()` for function setup. Check resource limits before allocations.
5. **Write tests** in `crates/zapcode-core/tests/`.
6. **Update bindings** if the feature affects the public API.

---

## Development commands

```bash
# Run all tests (214 tests)
cargo test

# Run benchmarks
cargo bench

# Check all crates
cargo check --workspace

# Lint
cargo clippy --workspace
```

---

## Testing philosophy

Every language feature must have:
1. A positive test (it executes correctly)
2. Edge case tests (boundary conditions, empty inputs)
3. A sandbox escape test where applicable

Tests live in `crates/zapcode-core/tests/`. Files are organized by feature:
`basic.rs`, `builtins.rs`, `classes.rs`, `generators.rs`, `async_await.rs`,
`control_flow.rs`, `functions.rs`, `integration.rs`, `objects_arrays.rs`,
`sandbox.rs`, `error_handling.rs`, `variables.rs`, `snapshot.rs`.

---

## Relationship to other projects

- **[pydantic/monty](https://github.com/pydantic/monty)** — direct inspiration, Python equivalent.
  Same architectural patterns (parse → compile → VM → snapshot).
- **[oxc](https://github.com/oxc-project/oxc)** — parser dependency. Don't replace it.
- **[boa](https://github.com/boa-dev/boa)** — full JS interpreter in Rust. Useful as reference. Not a dependency.

---
> Source: [TheUncharted/zapcode](https://github.com/TheUncharted/zapcode) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-22 -->
