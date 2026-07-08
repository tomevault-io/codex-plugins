# Project Guidelines

## Go Changes

- Prefer the Go standard library over new dependencies. If a new dependency is necessary, keep it narrowly justified and update modules with `go mod tidy`.
- Keep package responsibilities aligned with the existing layout: CLI wiring in `cmd/`, logging internals in `log/`, telemetry setup in `otel/`, and version metadata in `version/`.
- For CLI and configuration work, follow the existing Cobra and Viper pattern in `cmd/root.go`: define flags on commands, bind them through Viper, and avoid ad hoc environment or config parsing.

## Logging and Telemetry

- Use the local `log` package from application code instead of introducing direct `zap` usage outside the logging package internals.
- When passing a logger through execution flow, store and retrieve it via `log.AddToContext` and `log.GetFromContext`.
- Extend observability through `otel.SetupTelemetry` and the existing telemetry options rather than creating separate OpenTelemetry provider setup paths.

## Validation

- For meaningful Go changes, run the narrowest relevant validation first, then prefer `make test-suite` before finishing broader changes.
- Use `make lint` or `make codestyle` when edits can affect formatting or lint rules.
- Use `make test`, `make fast-tests`, or `make slow-tests` when working specifically on test behavior.

## Contributor Expectations

- Keep changes incremental and directly useful to the codebase.
- Avoid introducing project-wide setup changes, workflow changes, or new dependencies unless the task clearly requires them.
- When changing shared tooling such as the Makefile, lint config, pre-commit hooks, Dockerfile, or workflows, treat that as a higher-bar change and keep it tightly scoped.

---
> Source: [srlmgr/backend](https://github.com/srlmgr/backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-08 -->
