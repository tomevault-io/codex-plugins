# Jet – Copilot Onboarding Instructions

## What this repo does
- **Jet** is an LLVM-based JIT compiler for the Ethereum Virtual Machine (EVM)
- Rust workspace that JIT-compiles EVM bytecode to LLVM IR, then to native code via LLVM ORC JIT (`inkwell` bindings)
- Ideal for MEV use cases: same contract executed thousands of times with warm data
- Core value: native code execution vs. interpretation, with LLVM optimization passes

## Repository structure

### Workspace layout
- **Root files:**
  - `Cargo.toml` (workspace manifest with 4 members)
  - `Makefile` (build automation, exports LLVM env vars)
  - `.cargo/config.toml` (Android/Termux linker flags)
  - `rust-toolchain.toml` (pins stable Rust channel)
  - `README.md`, `DEVELOPMENT.md` (getting started, dev workflow)
  - `.nextest.toml` (cargo-nextest test runner config)

- **Workspace members (4 crates):**
  1. `crates/jet` - Main compiler crate
     - Edition 2024
     - Parses EVM opcodes, builds LLVM IR, drives JIT
     - Key modules: `builder/{contract,env,manager,ops}.rs`, `engine/mod.rs`, `instructions.rs`
     - Binary: `bin/jetdbg.rs` (debugger/executor)
     - Tests: `tests/test_roms.rs` (EVM opcode integration tests)
  
  2. `crates/jet_runtime` - Runtime execution context
     - Edition 2024
     - Crate type: `["dylib", "lib"]`
     - Builtins invoked from generated LLVM IR
     - Key modules: `{lib,exec,builtins,symbols}.rs`, `binding/mod.rs`
  
  3. `crates/jet_ir` - Shared LLVM IR types
     - Edition 2024
     - Function registry for Jet EVM JIT compiler
  
  4. `crates/jet_push_macros` - Procedural macros
     - Edition 2024
     - Proc-macro crate for test utilities

- **Scripts:** `scripts/{detect-llvm,detect-platform,install-llvm,llvm}.sh`
- **Documentation:** `docs/` (see [Key Documentation](#key-documentation) section)
- **CI:** `.github/workflows/ci.yml` (GitHub Actions, see [CI/CD](#cicd) section)

### Key source files
- `crates/jet/src/`:
  - `lib.rs` - Public API exports
  - `instructions.rs` - EVM instruction enum and opcode mappings
  - `builder/contract.rs` - Bytecode chunking, CFG construction, jump tables
  - `builder/ops.rs` - LLVM IR generation for each EVM opcode
  - `builder/env.rs` - Compilation environment (types, symbols, modes)
  - `builder/manager.rs` - Build orchestration
  - `engine/mod.rs` - LLVM ORC JIT engine, symbol resolution
  - `bin/jetdbg.rs` - CLI debugger for EVM contracts

- `crates/jet_runtime/src/`:
  - `builtins.rs` - Complex operations implemented in Rust (EXP, KECCAK256, etc.)
  - `exec.rs` - Execution context (stack, memory, storage, logs)
  - `symbols.rs` - Runtime symbol name constants
  - `runtime_builder.rs` - LLVM IR declarations for runtime functions

## Toolchain & dependencies

### Rust
- **Version:** Stable (pinned in `rust-toolchain.toml`)
- **Editions:** 2024 (jet, jet_runtime, jet_ir, jet_push_macros)
- **Components:** rustfmt, clippy (for CI)
- **No nightly required** - all operations use stable

### LLVM
- **Version:** 21 (REQUIRED)
- **Detection:** `scripts/detect-llvm.sh` finds `/usr/lib/llvm-21`
- **Environment:** Makefile exports `LLVM_SYS_211_PREFIX` automatically
- **Bindings:** 
  - `inkwell` - Git dependency with `llvm21-1-prefer-dynamic` feature
  - `llvm-sys` - Expects `llvm-config-21` or `LLVM_SYS_211_PREFIX`

### Dependencies
- **LLVM tooling:** `inkwell` (Git: TheDan64/inkwell, branch with LLVM 21 support)
- **Crypto:** `sha3`, `bnum` (256-bit arithmetic), `hex`
- **CLI:** `clap` (derive), `colored`, `syntect` (syntax highlighting)
- **Serialization:** `serde`, `serde_json`, `bincode`
- **Logging:** `log`, `simple_logger`
- **Error handling:** `thiserror`

### Platform support
- **Supported:** Linux (Debian/Ubuntu), macOS (darwin), Android (Termux)
- **Detection:** `scripts/detect-platform.sh` branches by platform
- **Termux-specific:** 
  - `.cargo/config.toml` contains linker flags
  - `CARGO_TARGET_DIR` relocated to `/data/data/com.termux/files/home/.cargo/jet-target`

## Build, test, lint workflow

### CRITICAL: Bootstrap LLVM first
**ALWAYS run this before any cargo command:**
```bash
make install-llvm
```

This:
- Detects platform via `scripts/detect-platform.sh`
- Installs LLVM 21 via apt (Debian/Ubuntu) or appropriate method
- May require `sudo` for system package installation
- Takes 1-3 minutes depending on network/disk speed

**Failure symptoms if LLVM missing:**
- `cargo check` fails with: `No suitable version of LLVM was found system-wide or pointed to by LLVM_SYS_211_PREFIX`
- `llvm-sys` build script cannot find `llvm-config-21`

**Verification:**
```bash
bash scripts/detect-llvm.sh  # Should output: /usr/lib/llvm-21
which llvm-config-21         # Should find executable
```

### Build commands
```bash
# Standard workflow
make check      # cargo check --all-targets --all-features (fastest validation)
make build      # cargo build
make fmt        # cargo fmt --all (auto-format)
make clippy     # cargo clippy --all-targets --all-features -- -D warnings (strict)

# Combined pre-push check
make commit-check  # Runs: fmt-check, check, clippy, test-all
make ci            # Alias for commit-check (mirrors CI pipeline)
```

**Note:** `make commit-check` is the **recommended pre-push command** - it runs all CI checks locally.

### Testing

**Test infrastructure:**
- Test runner: `cargo-nextest` (faster, better output than cargo test)
- Install: `make install-tools` or `cargo install cargo-nextest --locked`
- Config: `.nextest.toml`

**Test commands:**
```bash
make test         # cargo nextest run --all-features (fastest)
make test-cargo   # cargo test --all-features (fallback if nextest unavailable)
make doctest      # cargo test --doc --all-features (nextest doesn't support doctests)
make test-all     # Runs both: nextest + doctests
```

**Test structure:**
- `crates/jet/tests/test_roms.rs` - Main EVM opcode integration tests
  - Uses `rom_tests!` macro to define test cases
  - Each test: bytecode ROM → expected stack/memory/storage state
  - No external EVM node required - pure LLVM JIT execution
- `crates/jet/tests/invalid_opcode.rs` - Error handling tests

### Linting & formatting
```bash
# Check formatting (CI enforced)
make fmt-check    # cargo fmt --all -- --check

# Auto-format (CI will auto-commit this)
make fmt          # cargo fmt --all

# Lint (strict mode)
make clippy       # -D warnings (all warnings are errors in CI)

# Auto-fix clippy issues
make clippy-fix   # cargo clippy --fix
```

**Clippy notes:**
- CI enforces `-D warnings` (zero-warning policy)
- Common issues: unused variables, manual_div_ceil, needless borrows
- **CRITICAL:** Agents must NOT add `#[allow(...)]` pragmas without explicit permission. Fix the underlying issue or report why it should be allowed.

## CI/CD

### GitHub Actions workflow
- **Location:** `.github/workflows/ci.yml`
- **Triggers:** push to `master`/`main`, PRs to `master`/`main`/`ts/*`, manual dispatch
- **Runner:** `ubuntu-latest` (single sequential job)
- **Permissions:** `contents: write` (for auto-commit formatting fixes)

### CI pipeline steps (16 total)
1. Checkout code
2. **Cache LLVM 21** (keyed on Cargo.lock + install scripts)
3. **Restore LLVM from cache** (conditional: cache hit)
4. **Install LLVM 21** (conditional: cache miss, ~5 minutes)
5. **Log LLVM shared libraries** (verification step)
6. **Set LLVM env vars** (`LLVM_SYS_211_PREFIX`, include paths)
7. **Cache Rust artifacts** (Swatinem/rust-cache)
8. **Install Rust stable** (with rustfmt, clippy)
9. **Apply formatting fixes** (`cargo fmt --all`)
10. **Commit formatting fixes** (conditional: push or same-repo PR)
11. **Run clippy** (with `-D warnings`)
12. **Install cargo-nextest**
13. **Check all targets** (`cargo check`)
14. **Build** (`cargo build --verbose --all-features`)
15. **Run tests with nextest** (`--no-fail-fast`)
16. **Run doctests** (`cargo test --doc`)

### CI features
- **Smart caching:** LLVM binary (cached), Rust artifacts (swatinem)
- **Auto-formatting:** CI auto-commits `cargo fmt` changes and pushes back
- **Strict mode:** `RUSTFLAGS="-D warnings"` environment variable
- **Fast feedback:** Clippy before expensive build/test
- **No-fail-fast:** Tests continue after first failure (better error visibility)

### Environment variables
```bash
CARGO_TERM_COLOR=always    # Colored output
RUST_BACKTRACE=1           # Full backtraces on panic
CARGO_INCREMENTAL=0        # Disable incremental (CI optimization)
RUSTFLAGS="-D warnings"    # Treat warnings as errors
```

## Key documentation

**Architecture & design:**
- `docs/architecture.md` - Complete system architecture (compilation pipeline, memory model, CFG)
- `docs/bytecode-to-llvm-blocks.md` - Bytecode chunking, jump tables, CodeBlock lifecycle
- `docs/jet-description.md` - High-level project description
- `docs/manifesto.md` - Design philosophy

**Development guides:**
- `DEVELOPMENT.md` - Toolchain setup, workflow, CI details
- `docs/process/new-opcode.md` - **CRITICAL: Step-by-step opcode implementation guide**
  - Includes common pitfalls (endianness, stack order, zero division)
  - Required reading before implementing EVM opcodes
- `docs/process/segfault-troubleshooting.md` - Debugging and reporting segfaults (P0 priority)

**ADRs (Architecture Decision Records):**
- `docs/adrs/adr-001.md`, `adr-002.md`, `adr-003.md`, `adr-004.md` - Design decisions with rationale

**EVM spec references:**
- Use `evm-spec-lookup` skill if available for EVM opcode specifications
- If skill not available, note it and continue with implementation based on existing patterns

**Historical:**
- `docs/plans/2026-01-27-restore-build-system.md` - Build system restoration notes

## Development patterns & best practices

### Stack representation
- **Internal format:** Little-endian (32-byte words)
- **PUSH immediates:** Byte-reversed on load (EVM is big-endian)
- **Builtins:** Receive little-endian data, use `from_le_bytes()`
- **256-bit values:** Use `bnum::U256::from_digits([u64; 4])` with `u64::from_le_bytes()`

### Stack operations
```rust
// Always use helper functions (never manipulate stack directly)
let (a, b) = stack_pop_2(bctx)?;     // Returns (top, second)
stack_push_int(bctx, result)?;       // Push integer
stack_push_ptr(bctx, ptr)?;          // Push pointer
```

**CRITICAL:** `stack_pop_2()` returns `(top, second)` where `top` is most recently pushed.
For SUB: `PUSH 3; PUSH 10; SUB` → pops `(10, 3)` → computes `10 - 3 = 7` (NOT `3 - 10`).

### Division by zero (EVM semantics)
**EVM requirement:** Division/modulo by zero MUST return 0 (not undefined behavior).

```rust
// WRONG - LLVM poison value on zero divisor
let result = bctx.builder.build_int_unsigned_div(a, b, "div")?;

// ALSO WRONG - select still evaluates both arms, causing poison
let zero = bctx.env.types().i256.const_zero();
let b_is_zero = bctx.builder.build_int_compare(IntPredicate::EQ, b, zero, "b_is_zero")?;
let div_result = bctx.builder.build_int_unsigned_div(a, b, "div")?;
let result = bctx.builder.build_select(b_is_zero, zero, div_result, "final")?;
```

**CORRECT approach:** Use explicit branching (see below).

Applies to: DIV, MOD, SDIV, SMOD, and any division-like operations.

### LLVM poison avoidance
- **Branching pattern:** Use explicit `conditional_branch` for zero checks (not `select`)
- **Reason:** LLVM eagerly evaluates both arms of `select`, causing poison on div-by-zero
- **Example:** See DIV/SDIV/MOD/SMOD implementations in `builder/ops.rs`
- **Helper function:** Use `build_zero_guard()` helper for division-by-zero handling

```rust
// CORRECT: Create blocks for explicit branching
let zero_block = bctx
    .env
    .context()
    .append_basic_block(bctx.func, "zero");
let nonzero_block = bctx
    .env
    .context()
    .append_basic_block(bctx.func, "nonzero");
let cont = bctx
    .env
    .context()
    .append_basic_block(bctx.func, "cont");

// Conditional branch based on divisor
bctx.builder.build_conditional_branch(b_is_zero, zero_block, nonzero_block)?;

// In zero_block: push zero and jump to cont
bctx.builder.position_at_end(zero_block);
stack_push_int(bctx, zero)?;
bctx.builder.build_unconditional_branch(cont)?;

// In nonzero_block: compute and push result, jump to cont
bctx.builder.position_at_end(nonzero_block);
let result = bctx.builder.build_int_unsigned_div(a, b, "div")?;
stack_push_int(bctx, result)?;
bctx.builder.build_unconditional_branch(cont)?;

// Continue from cont
bctx.builder.position_at_end(cont);
```

### Memory operations
**Always expand memory before access:**
```rust
let offset_i32 = load_i32(bctx, offset_ptr)?;
let size = bctx.env.types().i32.const_int(SIZE, false);
bctx.builder.build_call(
    bctx.env.symbols().mem_expand(),
    &[bctx.registers.exec_ctx.into(), offset_i32.into(), size.into()],
    "expand",
)?;
```

- Rounds to 32-byte boundaries
- Updates `memory_len` monotonically
- Reallocates if needed
- **Must call before memory access** to prevent UAF (Use-After-Free)

### Builtin function pattern
**Rust side** (`crates/jet_runtime/src/builtins.rs`):
```rust
pub extern "C" fn jet_ops_exp(base_ptr: &mut [u8; 32], exp_ptr: &[u8; 32]) -> i8 {
    // Little-endian representation
    let base = U256::from_digits([...]);
    let exp = U256::from_digits([...]);
    let result = base.pow(exp);
    // Write result back to base_ptr
    0  // Success return code
}
```

**LLVM IR declaration** (`crates/jet_runtime/src/runtime_builder.rs`):
```rust
self.module.add_function(
    "jet.ops.exp",
    self.types.i8.fn_type(&[self.types.ptr.into(), self.types.ptr.into()], false),
    None,
);
```

**Symbol registration** (`crates/jet/src/builder/env.rs`):
```rust
pub(crate) struct Symbols<'ctx> {
    exp: FunctionValue<'ctx>,
}
```

**JIT linking** (`crates/jet/src/engine/mod.rs`):
```rust
map_fn(sym.exp(), builtins::jet_ops_exp as *const () as usize);
```

### Opcode implementation checklist

For complete implementation guidance, see `docs/process/new-opcode.md`. Key steps:

1. **Read EVM spec** - Use `evm-spec-lookup` skill if available; understand operand order and edge cases
2. **Define instruction** - Add to `Instruction` enum in `instructions.rs`
3. **Implement handler** - Add function to `builder/ops.rs`
4. **Route opcode** - Add case to match in `builder/mod.rs`
5. **Add builtin** (if complex) - Follow builtin function pattern above
6. **Write tests** - Cover basic case, edge cases, endianness, errors
7. **Test locally** - Run `make test` or `make test-cargo`
8. **Pre-push check** - Run `make commit-check` (fmt, clippy, tests)

## Troubleshooting

### LLVM not found
```bash
# Symptom
cargo check  # Error: No suitable version of LLVM was found

# Solution 1: Install LLVM
make install-llvm

# Solution 2: Verify detection
bash scripts/detect-llvm.sh  # Should output: /usr/lib/llvm-21

# Solution 3: Manual env var
export LLVM_SYS_211_PREFIX=/usr/lib/llvm-21
cargo check
```

### cargo-nextest not found
```bash
# Symptom
make test  # Error: no such command: `nextest`

# Solution: Install nextest
make install-tools
# Or: cargo install cargo-nextest --locked

# Workaround: Use cargo test instead
make test-cargo
```

### Tests segfault (SIGSEGV)

**CRITICAL:** Test segfaults are P0 priority issues that must be addressed immediately.

For complete troubleshooting guidance, see `docs/process/segfault-troubleshooting.md`.

**Quick reference:**
```bash
# Run with debug output
RUST_LOG=debug cargo test

# Run single test to isolate
cargo test --test test_roms -- test_name --exact

# Use jetdbg for interactive debugging
cargo run --bin jetdbg
```

**Required action:**
- **You MUST either:** Submit a PR that fixes the segfault, OR create a new issue with detailed reproduction steps and analysis
- Do not leave segfaults unaddressed or undocumented

### Incremental compilation cache issues
```bash
# Symptom: Tests fail after fixing implementation

# Solution: Clean rebuild
rm -rf target  # Or on Termux: rm -rf /data/data/com.termux/files/home/.cargo/jet-target
cargo test
```

### Clippy warnings in CI
```bash
# Symptom: CI fails on clippy step with warnings

# Solution: Fix locally
make clippy         # See warnings
make clippy-fix     # Auto-fix some issues

# CRITICAL: Do NOT add #[allow(...)] pragmas without explicit permission
# You must fix the underlying issue or report why it should be allowed
```

### Auto-formatted code not matching expectations
```bash
# The CI auto-commits formatting changes
# If you see unexpected formatting:

# 1. Pull latest from remote (CI may have pushed formatting fixes)
git pull

# 2. Run fmt locally before committing
make fmt

# 3. Check fmt before pushing
make fmt-check
```

### Platform-specific issues
**Termux (Android):**
- Requires `gcc-default` and `ndk-multilib-native-static` packages
- Install via: `make install-llvm` (handles platform detection)
- Target directory: `/data/data/com.termux/files/home/.cargo/jet-target` (not `./target`)

**macOS:**
- LLVM installed via Homebrew or official binaries
- Detection via `scripts/detect-platform.sh` → `darwin` branch

**Unsupported platforms:**
- `scripts/detect-platform.sh` will error with clear message

## Known issues & limitations

### Current known bugs (as of 2026-02-16)
All previously known bugs have been resolved. Any new segfaults or bugs discovered must be reported immediately as P0 issues.

### EVM opcode implementation status
- **Implemented:** Most arithmetic (ADD, MUL, SUB, DIV, MOD, ADDMOD, MULMOD, EXP, etc.)
- **Implemented:** Stack ops (PUSH*, DUP*, SWAP*, POP)
- **Implemented:** Memory ops (MLOAD, MSTORE, MSTORE8)
- **Implemented:** Control flow (JUMP, JUMPI, JUMPDEST)
- **Implemented:** Bitwise (AND, OR, XOR, NOT, BYTE, SHL, SHR, SAR)
- **Implemented:** Crypto (KECCAK256)
- **Not implemented:** External calls, contract creation, SELFDESTRUCT, etc. (see `instructions.rs`)

### Architecture limitations
- No gas accounting yet (planned future work)
- No external call support (CALL, DELEGATECALL, etc.)
- No contract creation (CREATE, CREATE2)
- No storage operations (SLOAD, SSTORE)
- No logging (LOG0-LOG4)
- Memory-based stack (not register-based) - suboptimal performance
- Jump tables built at compile time (dynamic jumps via table lookup) - suboptimal for dynamic contracts

## Additional resources

### External documentation
- [LLVM Documentation](https://llvm.org/docs/)
- [Inkwell (LLVM Rust bindings)](https://github.com/TheDan64/inkwell)
- [Ethereum Yellow Paper](https://ethereum.github.io/yellowpaper/paper.pdf)
- [EVM Opcodes](https://www.evm.codes/)
- [cargo-nextest](https://nexte.st/)

### Commit conventions
Follow conventional commits:
```
feat: implement OPCODE (0xHH)
fix: correct stack order in OPCODE
docs: update architecture documentation
test: add edge case for OPCODE
refactor: simplify LLVM IR generation
chore: update dependencies
```

### Getting help
- Review `docs/process/new-opcode.md` for detailed implementation guidance
- Check existing implementations in `builder/ops.rs` for patterns
- Reference builtin implementations in `crates/jet_runtime/src/builtins.rs`
- Use `evm-spec-lookup` skill if available for EVM opcode specifications

## Quick reference

### Essential commands
```bash
# One-time setup
make install-llvm          # Bootstrap LLVM 21 (required first)
make install-tools         # Install cargo-nextest (optional but recommended)

# Daily workflow
make fmt                   # Format code
make check                 # Fast validation
make clippy                # Lint
make test                  # Run tests (requires nextest)
make test-cargo            # Run tests (fallback, no nextest needed)
make commit-check          # Full pre-push validation

# Debugging
cargo run --bin jetdbg     # Run debugger/executor
RUST_LOG=debug cargo test  # Verbose test output
```

### File locations
```
crates/jet/src/builder/ops.rs          → Opcode implementations
crates/jet/src/instructions.rs         → Instruction enum
crates/jet_runtime/src/builtins.rs     → Complex operations (Rust)
crates/jet/tests/test_roms.rs          → Integration tests
docs/process/new-opcode.md             → Implementation guide
docs/process/segfault-troubleshooting.md → Segfault debugging (P0)
.github/workflows/ci.yml               → CI pipeline
```

### Environment variables
```bash
export LLVM_SYS_211_PREFIX=/usr/lib/llvm-21  # Manual LLVM path
export RUST_BACKTRACE=1                       # Full backtraces (default in Makefile)
export RUST_LOG=debug                         # Verbose logging
export CARGO_TARGET_DIR=/custom/path          # Override target directory
```

---

**Last updated:** 2026-02-16  
**LLVM version:** 21  
**Rust edition:** 2024 (jet, jet_runtime, jet_ir, jet_push_macros)  
**Test runner:** cargo-nextest (recommended) or cargo test (fallback)

---
> Source: [tcrypt25519/jet](https://github.com/tcrypt25519/jet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-04 -->
