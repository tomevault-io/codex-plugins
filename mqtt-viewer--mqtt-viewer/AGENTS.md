# GitHub Copilot Instructions for MQTT Viewer

## Project Overview

MQTT Viewer is a cross-platform desktop MQTT visualization and debugging tool built with:

- **Desktop Framework**: [Wails](https://wails.io/) v2.9.1 (embeds Go backend with web frontend)
- **Backend**: Go 1.22+
- **Frontend**: Svelte 5 + TypeScript 5.5 + Vite 5.4
- **Styling**: Tailwind CSS 3.4
- **Database**: SQLite with GORM

## Project Structure

```
mqtt-viewer/
├── backend/                    # Go backend services
│   ├── app/                    # Main application logic and API methods
│   ├── mqtt/                   # MQTT client manager (Paho MQTT)
│   ├── db/                     # Database layer (GORM + SQLite)
│   ├── models/                 # Data models (shared)
│   ├── protobuf/               # Protocol buffer support
│   ├── security/               # TLS/certificates
│   ├── topic-matching/         # MQTT topic pattern matching
│   ├── mqtt-middleware/        # Encoding/decoding middleware
│   ├── event-runtime/          # Go-to-frontend event system
│   └── logging/                # Logging infrastructure
├── frontend/                   # Svelte/TypeScript UI
│   ├── src/
│   │   ├── components/         # Reusable UI components
│   │   ├── views/              # Page/view components (Home, Connection, NewTab)
│   │   ├── stores/             # Svelte stores (state management)
│   │   ├── util/               # Utility functions
│   │   └── assets/             # Static assets
│   └── wailsjs/                # Auto-generated TypeScript bindings
├── events/                     # Event type definitions
├── main.go                     # Application entry point
├── wails.json                  # Wails configuration
└── justfile                    # Command shortcuts
```

## Development Commands

### Prerequisites

- Go 1.22+
- Node.js with pnpm (`npm install -g pnpm`)
- Wails CLI (`go install github.com/wailsapp/wails/v2/cmd/wails@v2.9.1`)
- Just (optional, recommended): https://github.com/casey/just

### Setup and Development

```bash
# Install dependencies
go mod tidy
cd frontend && pnpm install

# Start development server (hot reload enabled)
just dev
# or without Just:
wails dev
```

### Testing

```bash
# Run Go tests
just test PATH='./...'

# Run frontend tests
cd frontend && pnpm test

# Type checking
cd frontend && pnpm check
```

### Building

```bash
# Production build
just build VERSION="v1.0.0"

# Raspberry Pi build
just build-pi VERSION="v1.0.0"
```

### Database Migrations

```bash
# Create new migration (requires Atlas: https://atlasgo.io/getting-started/)
just new-migration NAME
```

## Architecture Patterns

### Backend (Go)

1. **Service Pattern**: Each backend module provides specific services (MQTT, DB, etc.)
2. **Manager Pattern**: `MqttManager` handles MQTT connection lifecycle
3. **Event-Driven**: Uses `events.ConnectionEvents` for loose coupling between backend and frontend
4. **Context-Based**: Go contexts for cancellation and timeouts
5. **Middleware Pattern**: MQTT middleware for message encoding/decoding

Example App struct pattern:

```go
type App struct {
    ctx            context.Context
    Db             *db.DB
    EventRuntime   *eventRuntime.EventRuntime
    AppConnections map[uint]*AppConnection
}
```

### Frontend (Svelte/TypeScript)

1. **Component-Based**: Organized by feature in `components/` and `views/`
2. **Reactive Stores**: Svelte stores in `stores/` for shared state
3. **View Routing**: Manual view switching (Home, Connection, NewTab)
4. **Type-Safe**: Strict TypeScript with `strictNullChecks: true`
5. **Path Aliases**: Use `@/` for imports from `src/`

Example store pattern:

```typescript
import { writable } from "svelte/store";

const connections = writable<ConnectionsState>({});
export default connections;
```

### IPC Bridge (Wails)

- Backend methods exposed via `wailsjs/go/` auto-generated bindings
- Frontend calls Go functions directly:

```typescript
import { GetConnections } from "wailsjs/go/app/App";
const connections = await GetConnections();
```

## Coding Conventions

### Go

- Use descriptive package names matching directory
- Keep exported methods in `app/` package for Wails bindings
- Use pointer receivers for struct methods
- Follow standard Go formatting (`gofmt`)

### TypeScript/Svelte

- Use TypeScript strict mode
- Prefer `@/` path alias for imports
- Use Svelte 5 reactivity patterns (e.g., `$state`, `$derived`, `$effect`)
- Follow component naming: PascalCase for components

### Tailwind CSS

- Use custom elevation classes: `elevation-0`, `elevation-1`, `elevation-2`
- Use custom color tokens: `primary`, `secondary`, `white-text`, `secondary-text`
- Dark theme by default

## Key Technologies

| Purpose | Library |
|---------|---------|
| MQTT Client | Eclipse Paho MQTT |
| UI Components | Melt UI |
| Icons | Phosphor Icons (phosphor-svelte) |
| Forms | Felte + Zod validation |
| Code Editor | CodeMirror 6 |
| Timeline | Vis.js Timeline |
| Animation | GSAP |

## Testing Guidelines

- Go tests use standard `testing` package
- Frontend tests use Vitest
- Test files: `*_test.go` (Go), `*.test.ts` (TypeScript)
- Run specific Go package tests: `just test PATH='./backend/app/...'`

## Important Notes

- Wails generates TypeScript bindings automatically during build
- Changes to Go methods require regeneration of bindings
- Frontend changes hot-reload quickly; Go changes require full rebuild
- Use `wails doctor` to diagnose Wails installation issues
- Maximum 10 concurrent MQTT connections supported

---
> Source: [mqtt-viewer/mqtt-viewer](https://github.com/mqtt-viewer/mqtt-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-19 -->
