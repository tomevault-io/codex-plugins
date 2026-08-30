# Copilot Instructions for bash-libcaller

This repository contains a high-performance Bash builtin (`callso`) for calling C functions via `libffi`.

## Critical Performance Guidelines

### 1. Avoid Subshells in Loops
**NEVER** use subshells `$(...)` inside the main game loop or high-frequency code paths.
*   **Bad:** `val=$(raylib IsKeyDown 65)` (Forks a process, ~100x slower)
*   **Good:** `callso -v val "$LIB_PATH" "bool i32" "IsKeyDown" 65` (Zero allocation, runs in-process)

The `callso -v variable_name` syntax binds the result directly to a Bash variable. Use this for all getters.

### 2. Type Signatures Matter
Be extremely precise with FFI types. Mismatches cause garbage data or crashes.
*   **`bool` vs `i32`:** Raylib's `bool` is 1 byte. If you use `i32` in the signature, `libffi` reads 4 bytes, potentially including garbage data from the register.
    *   **Correct:** `"bool i32"` for `IsKeyDown`
    *   **Incorrect:** `"i32 i32"`
*   **Structs:** Use the `{t1,t2}` syntax for structs passed by value.

### 3. Memory Management
*   **Structs:** `callso` handles struct-by-value returns by allocating memory.
*   **Pointers:** If a C function returns a pointer that needs freeing, you must manually call the free function (e.g., `UnloadSound`).
*   **Leaks:** The `callso` builtin tracks internal allocations for `ffi_type` construction. These are currently persistent for the session lifetime to support caching.

## Development Workflow

### Recompiling
Standard build (uses system libffi):
```bash
make clean && make
```

High-performance build (if custom libffi is available):
```bash
export PKG_CONFIG_PATH=$HOME/.local/libffi-custom/lib/pkgconfig
make clean && make
```

### Debugging
*   **Profiling:** `callso` writes stats to `/tmp/callso_profile` (call count, cache hits/misses, timing).
*   **Crash:** If the shell crashes, it's usually a signature mismatch or a segfault in the target library. Check `dmesg` or run bash inside `gdb`.

---
> Source: [GeorgLegato/bash-libcaller](https://github.com/GeorgLegato/bash-libcaller) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-30 -->
