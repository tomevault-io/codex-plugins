# Copilot Instructions — myworld

## Project Overview
A 2D world simulation game written in Go, rendered with [Ebiten v2](https://ebitengine.org/).  
Module: `github.com/x760591483/myworld`

## Architecture

```
core/          ← Pure simulation logic, zero external deps
api/           ← Planned: Commands / Events / Snapshots (all currently empty)
adapter/       ← Planned: http / local / websocket adapters (all currently empty)
frontend/
  ebiten/      ← Ebiten desktop renderer (game.go is the integration point)
  web/         ← Planned web frontend (empty)
cmd/
  desktop/     ← Entry point: Ebiten window (main runnable today)
  server/      ← Stub server
  replay/      ← Stub replay viewer
assets/        ← fonts/, sprites/
tools/debug/   ← Debug tooling (empty)
```

**Strict layering rule**: `core/` must never import `frontend/`, `adapter/`, or `api/`.  
`frontend/ebiten` is the only package that currently imports both `core/` and Ebiten.

## Entity Model (`core/`)

- **`Entity`** — base struct (ID uint64, X/Y float64, Radius, Health, Mass, Color)
- **`Creature`** embeds `Entity`; adds VelocityX/Y, Speed, Direction, eye geometry
- **`Plant`** embeds `Entity`; adds GrowthStage (static, no velocity)
- **`EntityType`** enum: `EntityTypeCreature`, `EntityTypePlant`, `EntityTypeObstacle`
- **`Color`** is a project-specific `struct{R,G,B uint8}`, **not** `image/color`
- IDs are assigned via `world.NextID()` (uint64 auto-increment, world-scoped)

```go
id := w.NextID()
c := core.NewCreature(id, x, y, radius, nil) // father *Creature — reserved for lineage
w.AddCreature(c)
```

## Simulation Loop

`World.Tick(dt float64)` in `core/tick.go` is the single simulation step.  
Currently it integrates `VelocityX/Y` into position; `core/physics.go` and `core/rules.go` are empty stubs awaiting implementation.

`frontend/ebiten/game.go` calls `w.Tick(1.0/60.0)` from `Game.Update()` at 60 TPS:
```go
ebiten.SetTPS(60)
// Game.Update() → w.Tick(1.0 / 60.0)
```

## Coordinate System

- World space: float64 X/Y, origin at centre of world
- Screen mapping (temporary, in `Game.Draw`): `sx = 320 + c.X`, `sy = 240 + c.Y`
- `camera.go` and `render.go` are empty stubs — camera/render logic goes there

## Build & Run

```powershell
# Run the desktop app (only working entry point today)
go run ./cmd/desktop

# Build everything (catches compile errors across all packages)
go build ./...

# Run tests
go test ./...
```

## Planned Expansion Points

| Path | Intended Purpose |
|---|---|
| `core/physics.go` | Collision detection, mass-based movement |
| `core/rules.go` | Eating, reproduction, death rules |
| `api/command.go` | Client→server commands |
| `api/event.go` | Server→client events |
| `api/snapshot.go` | Full world state snapshot for sync/replay |
| `adapter/local/` | In-process adapter (single-player, no network) |
| `adapter/http/` | HTTP polling adapter |
| `adapter/websocket/` | WebSocket real-time adapter |
| `frontend/ebiten/render.go` | Circle/eye rendering helpers |
| `frontend/ebiten/camera.go` | Camera transform (pan/zoom) |

## Key Conventions

- Drawing currently uses **rectangles as placeholders** for circles (`ebitenutil.DrawRect`); replace with proper circle rendering in `render.go`.
- `NewCreature` accepts `father *Creature` (currently unused) — preserve this parameter for future lineage/genetics features.
- `core/physics.go` and `core/rules.go` start as empty `package core` files — add functions there, do not create new files for those concerns.
- The `api/` layer is the intended boundary between core simulation and any network/replay transport.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/x760591483) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
