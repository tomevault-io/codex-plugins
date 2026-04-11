# Takhin Project - AI Coding Agent Instructions

## Project Overview
Takhin is a Kafka-compatible streaming platform rewritten in Go, consisting of two main components:
- **Takhin Core**: Kafka-compatible server (`backend/cmd/takhin/`)
- **Takhin Console**: Web management UI with REST API (`backend/cmd/console/`)

This is a monorepo with focus on backend Go development. Frontend React code exists in `projects/console/frontend/`.

## Architecture & Key Components

### Core Services Structure
```
backend/pkg/
├── kafka/          # Kafka protocol implementation
│   ├── server/     # TCP server handling Kafka connections
│   ├── handler/    # Request handlers (produce, fetch, metadata, etc.)
│   └── protocol/   # Binary protocol encoding/decoding
├── storage/        # Storage layer
│   ├── topic/      # Topic and partition management
│   └── log/        # Log segment storage
├── console/        # REST API server (Chi router)
├── coordinator/    # Consumer group coordination
├── raft/           # Raft consensus (no ZooKeeper dependency)
├── config/         # Koanf-based configuration (YAML + env vars)
├── logger/         # Structured logging (slog)
└── metrics/        # Prometheus metrics
```

### Component Interaction Pattern
1. **Kafka Handler** receives binary protocol requests → decodes using `pkg/kafka/protocol`
2. **Handler** validates and routes to backend (implements `Backend` interface)
3. **Backend** interacts with `topic.Manager` for storage operations
4. **Topic Manager** manages partitions as `log.Log` instances
5. **Console Server** provides HTTP REST API wrapping similar operations

Example: [handler.go](backend/pkg/kafka/handler/handler.go) shows this pattern - handlers instantiate with `config.Config` and `topic.Manager`, use protocol decoders/encoders for wire format.

### Configuration System
Uses [Koanf](https://github.com/knadh/koanf) with layered approach:
1. Load from `configs/takhin.yaml` 
2. Override with `TAKHIN_` prefixed env vars (e.g., `TAKHIN_SERVER_PORT`)
3. Validate and set defaults in [config.go](backend/pkg/config/config.go)

Example: `TAKHIN_STORAGE_DATA_DIR=/data` overrides `storage.data.dir` in YAML.

## Critical Development Workflows

### Building & Running
```bash
# Use Taskfile (not Makefile) - see Taskfile.yaml
task backend:build      # Builds to build/takhin
task backend:run        # Runs with configs/takhin.yaml
task backend:test       # Run all tests with race detector
task backend:test:unit  # Run only unit tests (no integration)
task backend:lint       # golangci-lint with timeout

# Manual runs for debugging
cd backend
go run ./cmd/takhin -config configs/takhin.yaml
go run ./cmd/console -data-dir /tmp/data -api-addr :8080
```

### Testing Patterns
- **Table-driven tests**: All handler tests follow this pattern (see [alter_configs_test.go](backend/pkg/kafka/handler/alter_configs_test.go))
- **Test setup**: Create temp dir with `t.TempDir()`, instantiate `topic.Manager`, then `handler.New()`
- **Protocol testing**: Encode request → pass to handler → decode response → assert
- **Use testify/assert**: `assert.NoError()`, `assert.Equal()`, etc.
- **Integration tests**: Tag with `// +build integration` (not yet implemented extensively)

Example test structure:
```go
func TestSomething(t *testing.T) {
    cfg := &config.Config{Storage: config.StorageConfig{DataDir: t.TempDir()}}
    mgr := topic.NewManager(cfg.Storage.DataDir, 1024*1024)
    handler := handler.New(cfg, mgr)
    // ... test logic
}
```

### Swagger API Documentation
Console API uses Swag for OpenAPI docs:
- Annotations in [main.go](backend/cmd/console/main.go) define metadata
- Handler functions have Swag comments (see [server.go](backend/pkg/console/server.go))
- Regenerate with `swag init -g cmd/console/main.go -o docs/swagger`
- Access at `/swagger/index.html` when running console

## Project-Specific Conventions

### Error Handling
- **Always wrap errors** with context: `fmt.Errorf("failed to X: %w", err)`
- **Never ignore errors**: No `_ = err` patterns
- Define sentinel errors for expected cases: `var ErrTopicExists = errors.New("topic exists")`

### Logging
- Use structured logging: `logger.Info("message", "key", value)`
- Component-specific loggers: `logger.Default().WithComponent("kafka-handler")`
- Log levels: debug → info → warn → error (configured via `logging.level` in config)

### Protocol Implementation
Kafka binary protocol is complex - always:
1. Check API version in request header
2. Use protocol encoder/decoder functions in `pkg/kafka/protocol/`
3. Return proper error codes (see `protocol.ErrorCode*` constants)
4. Write tests with actual binary data when possible

Example: [AlterConfigs handler](backend/pkg/kafka/handler/alter_configs_test.go) shows full encode → handle → decode cycle.

### Authentication (Console API)
- API key auth via `Authorization` header (supports both raw key and `Bearer <key>`)
- Configured with `-enable-auth` flag and `-api-keys` CSV list
- Health check and Swagger routes bypass auth
- See [auth.go](backend/pkg/console/auth.go) for implementation

## Common Tasks & Patterns

### Adding a New Kafka API Handler
1. Add request/response structs to `pkg/kafka/protocol/`
2. Implement encoder/decoder functions
3. Add handler method to `pkg/kafka/handler/handler.go`
4. Register in `Handle()` switch statement
5. Write table-driven tests in `*_test.go`

### Adding Console REST Endpoint
1. Add handler method to `pkg/console/server.go`
2. Add Swag comment above handler (see existing examples)
3. Register route in `setupRoutes()` method
4. Return JSON using `respondJSON()` helper
5. Update Swagger docs with `swag init`

### Configuration Changes
1. Add field to struct in `pkg/config/config.go`
2. Add to YAML schema with koanf tags
3. Update `setDefaults()` and `validate()` functions
4. Add to `configs/takhin.yaml` with comments

### Storage Operations
- Topics are created via `topicManager.CreateTopic(name, numPartitions)`
- Each partition is a `log.Log` with segments on disk
- Produce: `log.Append(batch)` → writes to active segment
- Fetch: `log.Read(offset, maxBytes)` → reads from segments
- See [manager.go](backend/pkg/storage/topic/manager.go)

## Code Quality Standards

### Go Conventions (Critical)
- Run `gofmt` and `goimports` (via `task backend:fmt`)
- Pass `golangci-lint` with no warnings (configured in `.golangci.yml`)
- 80%+ test coverage on new code
- Functions < 100 lines, cyclomatic complexity < 15
- Use pointer receivers for large structs or mutating methods

### Git Workflow
- Branch naming: `feature/`, `fix/`, `refactor/`, `test/`, `docs/`
- Commits follow Conventional Commits: `<type>(<scope>): <subject>`
  - Examples: `feat(kafka): add metadata v9 support`, `fix(console): auth middleware bypass`

### Documentation Location
All docs in `docs/` directory:
- `architecture/`: System design, component interaction
- `implementation/`: Feature implementation details
- `testing/`: Test strategy and coverage

## Known Gotchas
- **Binary protocol** is big-endian, must use `binary.BigEndian`
- **Coordinator** must be started before handler creation (`coord.Start()`)
- **Topic directories** are `<data-dir>/<topic>-<partition>` format
- **YAML keys** use dots (e.g., `kafka.broker.id`) but env vars use underscores after `TAKHIN_` prefix
- **Swag regeneration** required after adding/changing API annotations

## References
- Full architecture: [docs/architecture/](docs/architecture/)
- Kafka protocol: [docs/kafka-protocol.md](docs/)  
- Task commands: `task --list` or see [Taskfile.yaml](Taskfile.yaml)
- Config example: [configs/takhin.yaml](backend/configs/takhin.yaml)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/zinohome)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/zinohome)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
