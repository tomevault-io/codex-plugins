# Zig Project Instructions

## Project Context

This is a Zig project containing both a library module and an executable.

- **Language**: Zig (latest stable or nightly as configured in `build.zig`)
- **Build System**: Standard `zig build` system

## Architecture

- **Library (`src/root.zig`)**: Contains the core logic and reusable components. Exposed as the `zig_test` module.
- **Executable (`src/main.zig`)**: The entry point for the CLI application. Imports the `zig_test` module.
- **Build Logic (`build.zig`)**: Defines the build graph, including the library module, executable, and test steps.

## Build & Run Commands

- **Build**: `zig build` (installs to `zig-out/bin/zig_test`)
- **Run**: `zig build run` (builds and runs the executable)
- **Test**: `zig build test` (runs unit tests in both library and executable)
- **Fuzz Testing**: `zig build test -- --fuzz` (runs fuzz tests, see `src/main.zig`)

## Testing Conventions

- Use `std.testing` for assertions (e.g., `try std.testing.expect(...)`).
- Use `std.testing.allocator` for memory allocation in tests to detect leaks.
- Place unit tests in the same file as the code they test, using `test "description" { ... }` blocks.
- Fuzz tests are supported via `std.testing.fuzz`.

## Coding Conventions

- **Output**:
  - Use `std.debug.print` for debugging/logging (goes to stderr).
  - Use `std.fs.File.stdout` for actual program output.
- **Memory**: Explicitly manage memory. Use `defer` for cleanup.
- **Error Handling**: Use Zig's error union types (`!T`) and `try` keyword.

## Documentation & Tools

- **Zig Standard Library**: Use the `zig-docs` MCP tools to search and retrieve documentation for the Zig standard library and builtins.
  - `mcp_zig-docs_search_std_lib`: Search for std lib items by name to discover available types and functions (e.g., "ArrayList", "fs").
  - `mcp_zig-docs_get_std_lib_item`: Get detailed documentation, signatures, and examples for a specific item (e.g., "std.ArrayList").
  - `mcp_zig-docs_list_builtin_functions`: List all available builtin functions (prefixed with '@') to discover what is available.
  - `mcp_zig-docs_get_builtin_function`: Get detailed documentation and usage for a specific builtin function (e.g., "@import", "@memcpy").

## Key Files

- `build.zig`: Build configuration.
- `src/root.zig`: Library root.
- `src/main.zig`: Executable entry point.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rohit746) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
