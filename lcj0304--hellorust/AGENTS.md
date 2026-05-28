# Copilot Instructions - HelloWorld Rust Learning Project

## Project Overview

This is a Rust learning project demonstrating thread pool implementation patterns. The project uses **Rust 2024 edition** and focuses on concurrent programming fundamentals without external dependencies.

## Architecture

### Core Components

- **main.rs**: Application entry point with basic arithmetic operations and thread pool demonstration
- **threadpool.rs**: Custom thread pool implementation using `mpsc` channels and shared state with `Arc<Mutex<T>>`

### Thread Pool Design Pattern

The project implements a **bounded thread pool** with these key characteristics:

1. **Fixed worker threads**: Number of workers is set at creation time (`ThreadPool::new(size)`)
2. **MPSC channel for job distribution**: Uses `std::sync::mpsc` for thread-safe job submission
3. **Shared receiver pattern**: `Arc<Mutex<mpsc::Receiver<Job>>>` allows multiple workers to pull from single queue
4. **Graceful shutdown**: `Drop` trait implementation ensures workers finish before cleanup

```rust
// Typical usage pattern in this codebase:
let pool = ThreadPool::new(4);  // Create 4 worker threads
pool.execute(move || {
    // Task code here
});
```

## Key Conventions

### Error Handling
- Uses `assert!` for precondition validation (e.g., thread pool size > 0)
- Panics with `unwrap()` for operations expected to succeed in learning context
- No custom error types - this is intentional for learning simplicity

### Concurrency Patterns
- **Job type**: `Box<dyn FnOnce() + Send + 'static>` - one-shot closures that own their data
- **Worker lifecycle**: Workers loop until channel disconnects, then self-terminate
- **No graceful stop mechanism**: Workers shut down when `ThreadPool` is dropped

### Module Organization
- Single-level module structure: `mod threadpool` declared in main.rs
- Public API surfaces: `ThreadPool::new()` and `ThreadPool::execute()`

## Development Workflow

### Build and Run
```bash
cargo build          # Compile project
cargo run            # Run with thread pool demonstration
cargo check          # Fast compilation check without executable
```

### Edition Specifics
The project uses **Rust 2024 edition** (bleeding edge). If encountering compatibility issues:
- Check if your rustc version supports 2024 edition
- Fall back to `edition = "2021"` in Cargo.toml if needed

### No External Dependencies
This project intentionally uses only `std` library to demonstrate:
- Manual thread pool implementation (vs using `threadpool` crate)
- Core concurrency primitives (`thread`, `mpsc`, `Arc`, `Mutex`)

## When Modifying Code

### Adding Tasks to Thread Pool
- Ensure closures use `move` keyword to transfer ownership
- Tasks must be `Send + 'static` - cannot borrow local variables
- Example: `pool.execute(move || { /* capture by value */ })`

### Thread Pool Constraints
- Current implementation lacks:
  - Work-stealing optimization
  - Dynamic resizing
  - Graceful shutdown signal (workers stop only on drop)
  - Panic recovery in workers

### Future Enhancement Opportunities
If expanding this learning project:
- Add graceful shutdown with `Message` enum (Execute/Terminate pattern)
- Implement worker panic recovery
- Add metrics (completed tasks, queue length)
- Create builder pattern for configuration

## References

Key files for understanding the architecture:
- [src/threadpool.rs](../src/threadpool.rs) - Core thread pool implementation
- [src/main.rs](../src/main.rs) - Usage examples and demonstration code
- [Cargo.toml](../Cargo.toml) - Project configuration with Rust 2024 edition

---
> Source: [lcj0304/hellorust](https://github.com/lcj0304/hellorust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-28 -->
