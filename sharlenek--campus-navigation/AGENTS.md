<!-- Project-specific Copilot instructions for `campus-navigation` -->
# Copilot / AI Agent Instructions

This file contains concise, actionable guidance for an AI coding agent working in the `campus-navigation` C++ project. Focus on discoverable patterns, build/test workflows, and concrete file examples so agents can be productive immediately.

## Big picture
- **Purpose:** The project implements a campus navigation / student manager program. The runtime entrypoint is `src/main.cpp` which instantiates `CampusCompass` and calls `ParseCSV` and `ParseCommand`.
- **Major components:**
  - `src/CampusCompass.*`: high-level coordinator for CSV parsing and command handling.
  - `src/Graph.h`: graph data structure (locations, edges, shortest paths, MST, toggles).
  - `src/StudentManager.h`: placeholder for student-related logic (currently empty in repo).
  - `data/`: runtime CSV fixtures `edges.csv`, `classes.csv` consumed by `CampusCompass::ParseCSV`.
  - `test/`: Catch2-based tests in `test/test.cpp` (uses Catch2 v3 via CMake)

## Build & test workflow (practical)
- Primary build uses CMake. The project provides a `build/` dir and `CMakeLists.txt` that:
  - compiles two targets: `Main` and `Tests`
  - fetches Catch2 (v3.5.2) via `FetchContent`
  - copies `data/` into `build/data` via the `copy_data` target (so binaries open `data/...` at runtime)
- Recommended commands (cross-platform):
  - Configure and build:
    - `cmake -S . -B build`
    - `cmake --build build --config Debug` (or `Release`)
  - Run tests (Windows PowerShell):
    - `cd build; .\Tests.exe`  # run the test executable from `build`
  - Run the main program (from `build`):
    - `cd build; .\Main.exe` and supply input as required

Notes: CMake sets `-Wall -Werror` by default to match Gradescope; avoid removing `-Werror` unless necessary for local debugging because autograder expects it.

## Project-specific conventions & gotchas
- Test target must not include `src/main.cpp`. To test code, place testable code in `.h/.cpp` (e.g. `CampusCompass.cpp`) and list them in `add_executable(Tests ...)` — CMake already demonstrates this.
- CMake copies `data/` into the build directory. Use relative paths consistent with `main.cpp` (example uses `../data/edges.csv` when run from build directory). When running binaries from IDE, ensure working directory is `build/` or adjust paths.
- Do not expect `StudentManager.h` to contain implementation yet — it's currently empty; the agent may propose or scaffold it but must not assume existing behavior.
- `Graph` API surface is in `src/Graph.h` — functions to use/implement include `addLocation`, `addEdge`, `toggleEdge`, `edgeStatus`, `isConnected`, `shortestPath`, and `mstCost`.

## Tests and patterns to follow
- Tests use Catch2 and are discovered by `catch_discover_tests(Tests)` in `CMakeLists.txt`.
- Avoid including `main.cpp` in tests; use `CampusCompass.h` + `CampusCompass.cpp` for unit coverage.
- Tests often validate parsing/validation behavior: `CampusCompass::ParseCSV` returns `bool` to indicate success; `ParseCommand` returns `bool` for validation — tests expect these boolean semantics.

## Integration points & examples
- CSV parsing: `CampusCompass::ParseCSV(const string &edges_filepath, const string &classes_filepath)` opens files with `ifstream` (see `src/CampusCompass.cpp`). Implementations should handle header-skipping and `stoi` conversions carefully and return `false` on open/parse errors.
- Command parsing: `CampusCompass::ParseCommand(const string &command)` is the single entry for command dispatch; tests expect predictable boolean-return validation.
- Data files: `data/edges.csv` and `data/classes.csv` are canonical inputs for both runtime and test fixtures. Use them when writing example or integration tests.

## Where to look for examples
- `CMakeLists.txt` — shows how tests are wired, `copy_data` target, and Catch2 integration.
- `src/main.cpp` — canonical runtime input flow (call `ParseCSV` then read commands from stdin then call `ParseCommand`).
- `src/CampusCompass.cpp` — example of `ParseCSV` skeleton: open files, skip headers, parse CSV fields and convert with `stoi`.
- `src/Graph.h` — expected graph operations that `CampusCompass` will call.
- `test/test.cpp` — test file template showing Catch2 usage, input/output raw-string tests, and typical assertions.

## When editing or adding code, follow these rules
- Keep parsing logic in `CampusCompass` and graph algorithms in `Graph` (single responsibility).
- Add headers/implementation files to the `add_executable(...)` lists in `CMakeLists.txt` when required by CLion or local builds.
- Preserve `-Werror` until tests are passing locally; fixing warnings is preferred over disabling the flag.

## Minimal tasks an AI agent can do next
- Implement missing Graph methods or fill `StudentManager.h` scaffolding.
- Add unit tests in `test/` that exercise `ParseCSV` and `ParseCommand` boolean return semantics.
- Ensure new files are added to `CMakeLists.txt` and use `copy_data` semantics for any runtime CSV needs.

If anything here is unclear or you want more detail (examples of test cases, expected CSV schema, or preferred CLI behavior), tell me which area to expand and I'll update this file.

---
> Source: [sharlenek/campus-navigation](https://github.com/sharlenek/campus-navigation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-03 -->
