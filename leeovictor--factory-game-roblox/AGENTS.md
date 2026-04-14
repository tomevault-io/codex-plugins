<!-- .github/copilot-instructions.md - Guidance for AI coding agents -->

# Copilot / AI Agent Instructions

Short, actionable guidance to help an AI agent be productive in this repository.

- **Language & runtime**: this is a Roblox project written in Luau (Roblox Lua). Code lives under `src/` and runs inside Roblox Studio. Build and run with `rojo` (see `README.md`).

- **Big picture**: the project is an ECS-driven factory simulation inspired by Factorio. The client currently runs the simulation fully (multiplayer is not validated). Core pieces:
  - `src/Matter/` — the custom ECS implementation (World, Archetype, component helpers). Read `src/Matter/World.luau` and `src/Matter/component.luau` to understand entity lifecycle and immutable components.
  - `src/shared/Components.luau` — project component definitions; components are created with `Matter.component` and used in queries.
  - `src/client/` — client bootstrap (`init.client.luau`), UI modules (`ui/`), and systems (`client/systems/`). The main Loop is created in `init.client.luau`.
  - `src/server/` — minimal server bootstrap (`init.server.luau`) (currently not authoritative for simulation).

- **How simulation runs**: systems are scheduled on a `Loop` (see `init.client.luau`). Systems are either plain functions or tables exposing `{ system = fn, event = "FixedUpdate", after = {...} }`. Fixed-step logic uses `src/shared/FixedUpdate.luau` and the world exposes `FixedDeltaTime` on the client.

-- **Component & entity conventions**:

- This project customizes `Matter` to allow direct writes on component instances. Do NOT use `component:patch`; update component fields directly (for example: `comp.count += 1`).
- `world:insert` is reserved for special cases (creating new entities or adding components atomically during initialization). Typical runtime updates should mutate component instances returned by `world:query`.
- Component identity is still the component table (see `component.new` and `__len` id behavior). Add new components in `src/shared/Components.luau` and reference them from systems via `world:query(Components.SomeComponent)`.

- **Systems**:
  - Systems live under `src/client/systems/` (client) or `src/server/systems/` (server if added). Typical system module exports either a function or a table with `system`, `event`, and optional `after` keys — follow `src/client/systems/ProductionFacilitySystem.luau` as an example.
  - For fixed-timestep work, set `event = "FixedUpdate"` and rely on `world.FixedDeltaTime`.

- **Events & signals**: cross-module events use the `signal` package in `ReplicatedStorage.Packages.signal`. Shared event objects are in `src/shared/Events.luau`. Use `Events.ToolEquiped:Fire(...)` and `:Connect(...)` patterns.

- **UI & tools**: UI widgets and tool behavior are wired in `src/client/init.client.luau` and `src/client/ui/`. Tool attributes are validated at equip time — follow `GetToolAttributes` pattern in `init.client.luau` when adding tools.

- **Data specs**: item/recipe definitions live in `src/shared/ItemSpecs.luau` and `src/shared/RecipeSpecs.luau`. Use `ItemSpecs.ByKey[...]` and `RecipeSpecs` structures when working with production logic.

- **Packages & debugging**: third-party modules are under `Packages/`. The project uses `plasma` and a `Matter.Debugger` that integrates with it — see `init.client.luau` and `Matter.Debugger` usage for enabling visual debugging.

- **Build / run** (from repo root):
  - `rojo build -o "factory-game.rbxlx"` — build the place file.
  - Open `factory-game.rbxlx` in Roblox Studio and run `rojo serve` to sync live.

<!-- .github/copilot-instructions.md - Guidance for AI coding agents -->

# Copilot / AI Agent Instructions (concise)

This file contains the minimal, actionable knowledge an AI agent needs to be productive in this repository.

- Language & runtime: Luau (Roblox Lua). Source under `src/`. Build with `rojo` and run in Roblox Studio.

- Big picture: ECS-driven factory simulation (client-side authoritative). Key boundaries:
  - `src/Matter/` — custom ECS (World, Archetype, component helpers). Start with [src/Matter/World.luau](src/Matter/World.luau#L1) and [src/Matter/component.luau](src/Matter/component.luau#L1).
  - `src/shared/` — shared data: `Components.luau`, `ItemSpecs.luau`, `RecipeSpecs.luau`, `FixedUpdate.luau`.
  - `src/client/` — client bootstrap (`init.client.luau`), systems (`src/client/systems/`), UI (`src/client/ui/`). The simulation `Loop` is created in `init.client.luau`.

- How systems work: System modules export either a function or a table: `{ system = fn, event = "FixedUpdate", after = {"OtherSystem"} }`. Use `world.FixedDeltaTime` for fixed-step logic (see [src/shared/FixedUpdate.luau](src/shared/FixedUpdate.luau#L1)).

- Component & entity conventions (critical):
  - Components are mutated in-place. Do NOT use `component:patch` — instead update fields directly (`comp.count += 1`).
  - Use `world:query(Components.Some)` to iterate components: `for id, comp in world:query(Components.X) do ... end`.
  - `world:insert` is reserved for entity creation / atomic insertions during initialization — avoid using it for normal per-frame updates.

- Systems & ordering: Place systems in `src/client/systems/`. For ordering, use the `after` field; keep systems small and data-focused (see [src/client/systems/ProductionFacilitySystem.luau](src/client/systems/ProductionFacilitySystem.luau#L1)).

- Events & packages: Cross-module events use the `signal` package in `Packages/` and shared event objects in [src/shared/Events.luau](src/shared/Events.luau#L1). Use `:Fire(...)` and `:Connect(...)`.

- Data specs: Item and recipe definitions live in `src/shared/ItemSpecs.luau` and `src/shared/RecipeSpecs.luau`. Use `ItemSpecs.ByKey[...]` where appropriate.

- Debugging & tooling:
  - Visual debugger integration: `Matter.Debugger` + `plasma` (see `src/Matter/debugger/*` and `src/client/init.client.luau`).
  - Local packages live in `Packages/`. Packaging is managed via `wally` if adding new deps (see `wally.toml`).

- Build / run (from repo root):
  - Build place: `rojo build -o "factory-game.rbxlx"`
  - Serve and sync live to Studio: `rojo serve`

- Tests: Unit tests run inside a built place via `testez` and the helper `testez-companion-cli`.
  - Run: `testez-companion-cli` (from repo root). It will build/await check-ins and print a results summary.

- Files to inspect first when changing behavior:
  - Simulation logic: `src/client/systems/`
  - Component definitions: `src/shared/Components.luau`
  - ECS internals: `src/Matter/` (especially `World.luau`)
  - Bootstrap: `src/client/init.client.luau`

- Gotchas / constraints (do not change without care):
  - Direct component mutation is assumed across systems; replacing that pattern introduces tricky bugs.
  - Simulation runs client-side today — moving to server requires rethinking replication and event flows.
  - `FixedUpdate` includes an anti-spiral guard to avoid runaway fixed-step loops — review [src/shared/FixedUpdate.luau](src/shared/FixedUpdate.luau#L1) before changing timing logic.

If something above is unclear or you want examples (adding a new system, component, or UI wiring), tell me which area and I'll provide a concrete diff.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/leeovictor) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
