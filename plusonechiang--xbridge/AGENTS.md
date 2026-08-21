# Copilot Instructions

## Project

**xbridge** is a Rust project licensed under GPL-3.

## Build & Test Commands

```bash
# Build
cargo build

# Build release
cargo build --release

# Run tests
cargo test

# Run a single test by name
cargo test <test_name>

# Run tests in a specific module
cargo test <module>::

# Lint (Clippy)
cargo clippy -- -D warnings

# Format
cargo fmt

# Check formatting without applying
cargo fmt -- --check

# Mutation testing (if configured)
cargo mutants
```

## Notes

- This file should be updated as the project architecture, key conventions, and module structure take shape.

---
> Source: [PlusoneChiang/xbridge](https://github.com/PlusoneChiang/xbridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-21 -->
