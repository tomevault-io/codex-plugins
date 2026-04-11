# Copilot Instructions

Use `docs/technical/` as the primary source of truth for the repository's technical baseline. Read it before making architecture-affecting changes. Start with:

- `docs/technical/README.md`
- `docs/technical/baseline.md`
- `docs/technical/architecture.md`
- `docs/technical/repository-layout.md`
- then the specialized files relevant to the change

## Build, Test, and Lint

Use the standard Go toolchain as the source of truth. A thin `Makefile` may exist as a convenience layer only.

```sh
# Build the project
go build ./...

# Run the CLI
go run ./cmd/spotpilot <args>

# Run all tests
go test ./...

# Run a single test
go test ./path/to/package -run '^TestName$'

# Run lint checks
golangci-lint run
```

## High-Level Architecture

This repository is intended to be a Go CLI with a thin Cobra command layer over `internal/app`, `internal/domain`, and explicit boundary packages.

- Use `cmd -> internal/app -> internal/domain` as the core dependency direction
- Keep command registration centralized
- Let `internal/app` own use-case orchestration and consumer-owned ports
- Keep `internal/domain` free of transport/rendering concerns
- Let adapters in `internal/integrations` depend inward on app-owned ports
- Do not introduce a dedicated runtime package until concrete cross-cutting runtime concerns justify it

Avoid creating `pkg/` unless the project has a deliberate need for a reusable public package.

## Key Conventions

- Target Go `1.26`
- Module path is `github.com/ignatij/spotpilot`
- Official support is macOS and Linux
- Prefer the standard library first; add third-party dependencies only when they clearly earn their place
- Configuration should come from flags, environment variables, and an optional YAML config file
- Config precedence is `flags > env vars > config file > defaults`
- Use a centralized typed config loader
- Reserve the `SPOTPILOT_*` environment variable namespace
- Prefer `snake_case` for config keys and structured output field names
- Use XDG-style or otherwise platform-standard directories for config, cache, and state
- Default config filename is `config.yaml`
- Command results are JSON by default on `stdout`; use `--plain` for a concise human-readable line
- JSON is the primary stability target
- Use a standard envelope: `{\"ok\":true,\"command\":\"...\",\"state\":\"...\",\"message\":\"...\"}` with optional `result` payload
- Plain text output should be compact and derived from the same result model as JSON
- Human-oriented diagnostics belong on `stderr`
- Default output should stay quiet; use `--verbose` and `--debug` for additional detail
- Inject version, commit, and build date at build/release time and expose them through the `version` command
- Use `-ldflags` for build metadata injection
- Use stable exit codes and concise machine-parseable error messages
- Initial error categories are `validation`, `auth`, and `not_found`
- Treat no-op/idempotent outcomes as success with structured details, not as errors
- Commands should follow `load config -> validate inputs -> run use case -> render result -> map error/exit code`
- Map flags and resolved config into explicit input structs before the app layer
- Prefer deterministic non-interactive behavior by default
- Keep startup cheap and defer expensive work until command execution
- Prefer explicit flags over implicit current-directory behavior
- Use conservative defaults, explicit retries/timeouts, and consumer-owned ports/adapters for integrations
- Use Go's built-in `testing` package with table-driven tests
- Prefer env vars and OS-native keychain integration for credentials instead of plain-text secrets in config files
- Keep telemetry disabled by default; any future telemetry must be explicit opt-in

## Credential Storage

- Credentials are stored in the OS keychain (macOS Keychain / Linux secret-service) with a protected-file fallback when no keychain is available

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ignatij)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/ignatij)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
