# Messagarr AI Coding Instructions

## Architecture Overview

Messagarr is a Go notification aggregation service following a layered architecture:

```
cmd/messagarr/main.go     → Entry point, server lifecycle, graceful shutdown
internal/api/             → HTTP handlers, routing, Prometheus metrics
internal/channels/        → Channel dispatchers (SMTP, Discord, Slack, Teams)
internal/config/          → YAML config with env var interpolation (${VAR})
internal/models/          → Shared request/response types
internal/resilience/      → Retry, rate limiting, deduplication
internal/logger/          → Structured logging (slog), JSON in production
```

**Key Flow**: `POST /notify` → Deduplicator → Priority Router → Channel Dispatcher (parallel goroutines with retry) → External Services

## Development Commands

```bash
make build     # Build with swagger docs regeneration (ALWAYS run this, not raw go build)
make test      # Run tests with race detector
make docs      # Regenerate Swagger docs (runs automatically with build)
make lint      # Run golangci-lint
make coverage  # Generate coverage.html report
```

**Running locally**: Requires `.env` file with secrets, then `source .env && ./messagarr`

## Code Patterns

### Channel Dispatcher Interface

All channel implementations must satisfy:

```go
type Dispatcher interface {
    Send(req *models.NotificationRequest) error
    Name() string
}
```

See [internal/channels/discord.go](internal/channels/discord.go) as the reference implementation.

### Config with Environment Variable Interpolation

Secrets use `${VAR_NAME}` syntax in YAML, resolved at load time via `interpolateEnvVars()` in [internal/config/config.go](internal/config/config.go#L151-L165).

### Resilience Patterns

- **Deduplication**: SHA256 hash of notification content, TTL-based cache
- **Retry**: Exponential backoff (1s, 2s, 4s, max 30s), 3 retries max
- **Rate Limiting**: Token bucket, 10 req/sec per channel

### Testing Pattern

Tests use `httptest.NewServer` for HTTP handlers and mock configs. See [internal/api/server_test.go](../internal/api/server_test.go) for examples.

### Swagger/OpenAPI

Annotations live in handler functions using `// @` comments. Run `make docs` or `make build` to regenerate.

## Conventions

- **Logging**: Use `log/slog` (not log package). JSON output when `ENV=production`
- **Error wrapping**: Always use `fmt.Errorf("context: %w", err)`
- **Channel names**: Default to type (e.g., "discord") unless `name` field is specified
- **Priorities**: `high`, `normal`, `low` map to channel groups in config
- **Version**: Set via ldflags from `VERSION` file, not hardcoded

## File Organization

- New channel types → `internal/channels/{type}.go` implementing `Dispatcher`
- New API endpoints → `internal/api/server.go` with Swagger annotations
- Configuration changes → Update `internal/config/config.go` structs + validation
- Models shared across packages → `internal/models/models.go`

---
> Source: [eslutz/Messagarr](https://github.com/eslutz/Messagarr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-07 -->
