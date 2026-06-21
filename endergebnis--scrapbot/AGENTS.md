# Copilot Instructions

## Performance Target

**RAM budget: <200 MB** at steady state under load. **CPU: minimize cycles in every hot path.**
Every allocation, clone, and indirection must be justified. When in doubt, benchmark — not guess.

## Quick Start

```bash
cp .env.example .env   # then edit .env with your DISCORD_TOKEN
cargo run --release
```

## Build, Test, and Lint

```bash
# Release build (LTO + single codegen unit for max inlining)
cargo build --release

# Dev build
cargo build

# Run all tests
cargo test

# Run a single test
cargo test <test_name>

# Lint (treat warnings as errors)
cargo clippy -- -D warnings

# Format check
cargo fmt -- --check

# Benchmark (requires nightly)
cargo +nightly bench

# Profile with samply / perf
samply record -- cargo run --release
```

## Cargo.toml Release Profile

```toml
[profile.release]
opt-level = 3
lto = "fat"
codegen-units = 1
panic = "abort"
strip = "symbols"
```

## Architecture

Async Discord bot (tokio + Serenity) with a **zero-cost enum dispatch plugin system**. `bot.rs` holds the core event loop and dispatches interactions via `ModuleKind` enum matching — no vtable, no boxing. Modules are compiled statically.

**Module dispatch** (`modules/mod.rs`):
- `ModuleKind` enum with one variant per module. New modules add a variant + match arm.
- Each module struct has `name()`, `validate()`, `handles()`, `register_commands()`, `execute()`.
- Startup validates all modules; failures are **skipped with a logged error**, never crash the bot.

```
src/
  main.rs           # tokio (2 workers, 2MiB stack), mimalloc, .env loading
  bot.rs            # event loop & ModuleKind dispatch
  modules/
    mod.rs           # ModuleKind enum
    ping.rs          # example module
```

## Performance Rules (non-negotiable)

### Allocations
- **No `.clone()` in hot paths.** Borrow first, `Cow<str>` if ownership is conditional.
- **Prefer stack over heap.** Use `arrayvec` / `smallvec` for collections with known small upper bounds. All Discord embed fields, command option names, and interaction IDs fit in stack-allocated small strings.
- **Pre-allocate.** Any `Vec` or `String` that grows must call `.with_capacity()` with a tight estimate. Use `Vec::reserve_exact` before bulk inserts.
- **String constants are `&'static str`.** Never allocate `String` for static text (command names, descriptions, embed titles, error messages).

### Dispatch
- **Static dispatch over dynamic dispatch where feasible.** If the module set is known at compile time, prefer enums + `match` or generics over `Box<dyn Module>`. Only use trait objects if the module list must be extensible at startup.
- **Inline small functions.** `#[inline]` on any function under ~20 instructions in the interaction pipeline. Let LTO handle cross-crate inlining.

### Data
- **Zero-copy deserialization.** Gateway payloads should borrow from the receive buffer. Use `serde(borrow)` and `Cow<str>` on event structs. Avoid allocating a new `String` for every message content, username, or channel name.
- **`bytes::Bytes` for shared buffers.** If the same payload blob passes through multiple handlers, use `Bytes` (ref-counted, cheap clone) instead of `Vec<u8>` or `String`.

### Concurrency
- **Tokio runtime: tune worker threads.** Start with `worker_threads = 2` — Discord bots are I/O bound, not CPU bound. Monitor and adjust.
- **`tokio::spawn` is not free.** Don't spawn a task per interaction. Batch or use a channel-based work queue if parallelism is needed.
- **`Arc` sparingly.** Prefer `&` references with proper lifetimes. If shared state is unavoidable, use `Arc<tokio::sync::RwLock>` or lock-free structures (`AtomicBool` for feature flags).

### Memory Layout
- **`#[repr(C)]` or `#[repr(transparent)]`** on types crossing FFI boundaries or where layout matters for cache.
- **Keep hot structs small.** The interaction handler struct should fit in a single cache line (64 bytes). Audit with `std::mem::size_of`.
- **Use `Box` for large variants** in enums to avoid bloating the discriminant.

### Allocator
- **Use `mimalloc` or `jemalloc`.** Add to `Cargo.toml` and set as global allocator. mimalloc is typically faster for short-lived allocations; jemalloc gives better long-running RSS. Profile both.
```toml
[dependencies]
mimalloc = "0.1"
```
```rust
#[global_allocator]
static GLOBAL: mimalloc::MiMalloc = mimalloc::MiMalloc;
```

### Logging
- **`tracing` over `log`.** Structured, async-aware, and can be compiled to no-ops per crate.
- **Rate-limit diagnostics.** Use `tracing::instrument` on hot handlers but keep verbosity at `DEBUG`/`TRACE` for prod.

## Key Conventions

- **Slash commands only.** No prefix-based commands. Use Serenity's application command API.
- **Embeds for all responses.** No plain-text messages. Embed builder methods should accept `&str`, not `String`.
- **Static modules only.** Compiled into the binary — no `.so`/`.dylib`.
- **Graceful degradation.** Gateway reconnect handled. Failing modules skip, never crash the bot.
- **Prefer Serenity** over Twilight unless a concrete perf reason forces the switch.

---
> Source: [endergebnis/scrapbot](https://github.com/endergebnis/scrapbot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-21 -->
