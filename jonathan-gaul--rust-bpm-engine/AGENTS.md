## Purpose
Make AI coding agents productive immediately in this repository: a small Rust Proof-of-Concept BPM (workflow) engine. Focus on how processes, nodes, and task handlers are wired, where to change behavior, and the conventions for variable mapping and command parsing.

## Big picture (quick)
- Core executable: `src/main.rs` — contains a small `ProcessEngine`, `ProcessBuilder` helpers, and runnable examples.
- Workflow model: `src/tasks` — key pieces:
  - `node.rs` — `Node` and `NodeType` (Start, Task, ExclusiveGateway, ParallelGateway, End).
  - `config.rs` — `TaskConfig` holds command/http/docker settings and `input_mappings`/`output_mappings`.
  - `context.rs` — `TaskContext` (process variables passed to handlers).
  - `handlers.rs` + `handlers/*` — `TaskHandler` trait and concrete handlers (command, function).
  - `output.rs` — `TaskOutput` shape returned by handlers.

Design intent: a minimal engine that keeps process definitions in-memory, runs example processes from `main`, and demonstrates forks/joins, exclusive gateways, and command-based task handlers.

## Execution flow (what to edit to change behavior)
- `ProcessEngine::deploy` / `start_process` / `execute_instance` in `main.rs` drive execution. To change scheduling, orchestration rules, or persistence, start here.
- Node execution uses `NodeType::Task { handler_type, config }`. `handler_type` selects the handler implementation found in `src/tasks/handlers/*`.
- Task outputs are mapped back into process variables using each node's `TaskConfig.output_mappings` (task_output_var -> process_var).

## Important implementation details & conventions
- Command handler variable substitution: `handlers/command.rs` replaces args of the form `${varname}` with the process variable value from `TaskContext`.
- Command handler output parsing: stdout is parsed for variable lines using two heuristics:
  - Lines starting with `VAR:` are interpreted as `VAR:key=value` and extracted.
  - Lines containing `=` with no space before `=` (simple key=value) are parsed.
  - In addition the handler always exports `stdout`, `stderr`, and `exit_code` into the task variables map.
- Placeholder tasks: when a `TaskConfig` for a command task has `command = None`, the engine treats it as a placeholder and skips execution (see the check in `main.rs`).
- Parallel gateway semantics: forks create additional tokens (increase `active_tokens`); joins use `join_counters` to wait until all incoming tokens arrive before continuing.
- Error handling: handlers return `Result<TaskOutput, String>` or set `TaskOutput.success = false`. The engine logs failures but continues basic flow; update `execute_instance` to change failure policies.

## Patterns and examples (copyable)
- Build a command task and map output to process variables (see `main.rs` examples):

  let mut builder = ProcessBuilder::new("command_process")
      .add_start("start")
      .add_command_task("list_files", "List Files", "ls", vec!["-la"])
      .set_output_mapping("list_files", "stdout", "file_list");

- Use `${var}` substitution in `TaskConfig.args` to inject process variables into command arguments (handler does a simple replace).
- To return multiple variables from a command task, print lines like `VAR:key=value` or `key=value` (no spaces before `=`) from the program; `handlers/command.rs` will parse them.

## Where to look for change impact
- Add a new handler type: update `tasks::handlers::TaskHandlerType`, implement the trait in `src/tasks/handlers/<your>.rs`, and wire selection where handlers are invoked in `main.rs`.
- Persisting process instances: engine stores everything in-memory in `ProcessEngine` (see the `definitions` and `instances` HashMaps). Replace with DB calls where `deploy`, `start_process`, `execute_instance`, and `advance_token` interact with state.

## Build / run / debug
- Build: `cargo build` (Rust 2024 edition, see `Cargo.toml`).
- Run examples (prints execution traces): `cargo run`.
- Enable backtraces: in PowerShell use `$env:RUST_BACKTRACE = 1; cargo run`.
- The `main` program contains several runnable examples (sequential, parallel, conditional, command output parsing). Use or adapt them to test engine changes quickly.

## Tests
- This PoC has no test suite in the repo. When adding tests, follow Rust conventions and add unit tests near the code (`#[cfg(test)]`) or integration tests under `tests/`.

## Quick tips for AI agents
- Prefer changing small, focused functions: e.g., modify `handlers/command.rs` to change parsing or variable substitution.
- Preserve existing log printlns: they are intentionally used as a lightweight trace for debugging during refactor.
- When updating public shapes (`TaskConfig`, `TaskOutput`, `TaskContext`), update usages across `main.rs` and `handlers/*` — the repo is small so changes touch few files.

## Key files reference
- `src/main.rs` — engine loop, `ProcessBuilder`, and examples.
- `src/tasks/node.rs`, `src/tasks/config.rs`, `src/tasks/context.rs` — model shapes.
- `src/tasks/handlers.rs`, `src/tasks/handlers/command.rs` — handler trait and shell command implementation.
- `src/tasks/output.rs` — `TaskOutput` format.

If any section is unclear or you'd like more examples (e.g., a small unit test or a new handler stub), tell me which part to expand and I'll iterate.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jonathan-gaul)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/jonathan-gaul)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
