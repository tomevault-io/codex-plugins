# Copilot Instructions

## Build & Test

```bash
# Run all tests
go test -v

# Run tests with race detection and coverage (CI command)
go test -v -race -coverprofile=coverage.txt -covermode=atomic

# Run a single test
go test -v -run TestOtelRoundTripper_RoundTripWithTimeout

# Lint (requires golangci-lint)
golangci-lint run
```

## Architecture

This is a single-package Go library (`package otelroundtripper`) that implements `http.RoundTripper` to automatically emit OpenTelemetry metrics for HTTP client calls.

**Core components:**
- `round_tripper.go` — The `otelRoundTripper` struct implementing `http.RoundTripper`. The `RoundTrip` method orchestrates metric emission via hook methods (`beforeHook`, `afterHook`, `errorHook`, `successHook`, `failureHook`, `redirectHook`).
- `option.go` — Functional options pattern using an `Option` interface with `optionFunc` adapter.
- `config.go` — Internal `config` struct with defaults (name: `"http.client"`, parent: `http.DefaultTransport`).

**Flow:** `New(options...)` → builds config → creates metric instruments (counters/histograms) prefixed with the configured name → returns `http.RoundTripper`. Each `RoundTrip` call increments in-flight, delegates to parent transport, then records duration/success/failure/redirect metrics with OTel attributes.

## Conventions

- **Functional options pattern**: New options are added as `With*` functions in `option.go` returning `Option`.
- **Metric instrument creation**: Uses `must*` helper functions that panic on error — metrics are created once at initialization, not per-request.
- **Test structure**: Tests use `// Setup`, `// Arrange`, `// Act`, `// Assert`, `// Teardown` comment sections. Use `httptest.Server` via the `makeTestServer` helper for HTTP testing.
- **Formatting**: Uses `gofumpt` (stricter gofmt) via pre-commit hooks. Run `gofumpt -w .` to format.
- **OTel imports**: The metric API is aliased as `api "go.opentelemetry.io/otel/metric"` and semconv uses `semconv "go.opentelemetry.io/otel/semconv/v1.18.0"`.

---
> Source: [NdoleStudio/go-otelroundtripper](https://github.com/NdoleStudio/go-otelroundtripper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-24 -->
