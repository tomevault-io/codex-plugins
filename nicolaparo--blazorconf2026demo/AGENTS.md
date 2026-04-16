# BlazorConf2026Demo — Copilot Instructions

## Project Context

This is a **2D game engine built entirely in C#/Blazor** using HTML5 Canvas for rendering. The solution contains:

- **BlazorConf2026Demo.GameEngine** — Abstract game engine library (Blazor components + Canvas2D interop)
- **BlazorConf2026Demo.JumpGame** — Multiplayer platformer game using the engine
- **BlazorConf2026Demo.DemoGame** — Separate demo project (may have unrelated build errors)

**Target framework**: .NET 10.0 | **Rendering**: Blazor Server with interactive server-side rendering

## Architecture (Non-Standard — Read Carefully)

### Game Loop
The game loop is **NOT** a typical Blazor component lifecycle. It works via:
1. JS `requestAnimationFrame` + `setTimeout` (FPS limiting) in `gameinterop.js`
2. JS calls C# `OnAnimationFrameAsync()` via `[JSInvokable]` on `GameView.razor.cs`
3. C# batches all Canvas2D draw calls between `BeginBatchAsync()`/`EndBatchAsync()` — sent as single JS interop call
4. Loop: JS → C# frame → JS schedules next frame

### Class Hierarchy
- `GameEntity` (abstract base) → `Game` (game-level logic) and `GameObject` (instances)
- Both `Game` and `GameObject` share the same input API via `GameEntity`
- Game objects are created via `ActivatorUtilities.CreateInstance<T>()` — they support constructor injection

### Input System
- **Double-buffered**: `currentState` + `previousState` dictionaries, updated each frame
- Enables `Check` (held), `CheckPressed` (just pressed), `CheckReleased` (just released)
- Three sources: Keyboard, Mouse, Controllers (mobile devices)
- Controller input strings: `"LEFT"`, `"RIGHT"`, `"UP"`, `"DOWN"`, `"JUMP"`, `"RUN"`

### Multiplayer System
- `GameControllerHub` is registered as **Singleton** — shared across all Blazor circuits
- Mobile phones connect via `/gamecontroller` page, each gets a unique `Guid`
- Big screen shows lobby at `/` with QR code → starts game with all connected controllers
- Each `Player` instance binds to a specific controller via `Player.ControllerId`
- Camera follows the average X position of all players

### Rendering
- `GameView.razor` renders ALL game assets as hidden `<img>` tags (for Canvas `drawImage`)
- SVG `viewBox` + `foreignObject` provides viewport/camera system
- Pixel art: use `image-rendering: pixelated` on GameView

### DI Registration Pattern
- Assets: registered 3 ways (concrete, base `SpriteAsset`/`ImageAsset`, `IGameAsset` interface)
- Auto-discovery: `AddGameResources(assembly)` finds all sprite/image subclasses
- Game: `AddGame<TGame>()` registers as concrete + `Game` base

## Memory System

When resuming work on this project, check `memory.md` in the solution root for detailed state including:
- All files modified and why
- Implementation status of features
- Known issues and architectural gotchas

Update the memory file after completing significant changes.

## Coding Conventions

- Game objects go in `JumpGame/Objects/`
- Asset declarations go in `JumpGame/Assets/`
- Models (data classes) go in `JumpGame/Models/`
- Services go in `JumpGame/Services/`
- Pages go in `JumpGame/Components/Pages/`
- Extension methods go in `JumpGame/Extensions/`
- Use `record` for assets (e.g., `public record MySprite() : SpriteAsset(...)`)
- Tile IDs: `-1` = empty, `16/24/30` = non-solid special, `22` = egg placeholder
- Level files: `Levels/*.lvl.json`

## Build & Run

```bash
cd BlazorConf2026Demo.JumpGame
dotnet run
```

Game: https://localhost:7004 | Controller: https://localhost:7004/gamecontroller

For LAN multiplayer, bind to `0.0.0.0` or the machine's LAN IP in `Properties/launchSettings.json`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nicolaparo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-11 -->
