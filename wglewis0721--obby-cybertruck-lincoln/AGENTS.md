# GitHub Copilot Instructions — Obby-Cybertruck-Lincoln

This is a Roblox DRIVING SIMULATOR built in Luau. Not a racing game.

## Always
- --!strict at line 1 of every script
- Header: -- SCRIPT: [name] | LOCATION: [path] | SIDE: [Server/Client/Shared]
- Services via game:GetService() — never game.Players, game.Workspace globals
- task.wait() not wait() | task.spawn() not spawn() | task.delay() not delay()
- player.UserId as dictionary keys — never player.Name
- PascalCase: services, RemoteEvents, module export tables
- camelCase: local variables, functions, parameters
- LOUD_SNAKE_CASE: module-level constants
- Guard clauses for nil checks — fail early with warn(), not deeply nested ifs

## Never
- BodyVelocity or BodyGyro — use LinearVelocity / AngularVelocity constraints
- Put game logic (physics, DataStore, purchases) in LocalScripts
- Create a second ProcessReceipt handler — only one exists in PaintShopHandler.server.lua
- Overwrite entire PlayerData table — always read → modify specific field → write back

## Script Placement
- Server logic      → ServerScriptService/
- Shared modules    → ReplicatedStorage/Module/
- Top-level events  → ReplicatedStorage/ (direct children)
- Events folder     → ReplicatedStorage/Events/
- Client UI         → StarterGui/
- Client scripts    → StarterPlayer/StarterPlayerScripts/Client/

## Key Project Rules
- DataStore: PlayerData_v1 | Key: Player_[UserId]
- Vehicles named Vehicle_[UserId] in Workspace
- Checkpoints named Checkpoint_N inside map folders
- VehicleData Id = number | MapData Id = string — never mix

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/WGLewis0721)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/WGLewis0721)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
