# InterstellarOutpost.dx12 – AI Agent Working Notes

These instructions make AI coding agents productive in this Windows/MSVC C++23 game project.

## Context
- **Project:** Interstellar Outpost – a DirectX 12 game ported from an OpenGL codebase
- **Language:** C++23 (MSVC), uses WinRT for platform integration
- **Build:** Visual Studio solution (`InterstellarOutpost.slnx`), MSBuild via vcxproj files
- **Architecture:** Modular static libraries with raw-pointer cross-references to a singleton `App` (`g_app`)

## Architecture Overview

### Library Structure
| Directory | Purpose | Key Globals |
|-----------|---------|-------------|
| `NeuronCore/` | Platform abstractions, math, file I/O, OpenGL→DX12 translation layer | `g_prefsManager`,`g_windowManager` |
| `NeuronClient/` | Input, sound, networking, Eclipse UI framework | `g_inputManager`, `g_eventHandler` |
| `GameLogic/` | Game entities (`Entity`, `Building`, `WorldObject` hierarchies), AI, UI windows | — |
| `GameRenderer/` | Client render logic, responsible for rendering game entities and scenes | — |
| `InterstellarOutpost/` | Main application loop, renderer, camera, location management | `g_app` (singleton `App*`) |

### Data Flow
1. `main.cpp` owns the game loop (`UpdateAdvanceTime()`, `HandleCommonConditions()`)
2. `App` orchestrates subsystems: `Renderer`, `SoundSystem`, `Location`, `GlobalWorld`, `Server`/`ClientToServer`
3. Entity types derive from `WorldObject` → `Entity` (or `Building`) with type enums for polymorphic dispatch
4. Preferences managed via `g_prefsManager->GetInt/SetInt/Save()` pattern

## Formatting & Style
Enforced via `.editorconfig`:
- **Indent:** 2 spaces for C++ files
- **Line length:** 140 max
- **Braces:** same-line for simple blocks
- **New file names:** `UpperCamelCase` (e.g., `TeamControls.h`, `GameRenderer.cpp`). Legacy `snake_case` files are not renamed, but all newly created files must use `UpperCamelCase`.

## Patterns to Follow
- **RAII everywhere**: `std::unique_ptr` for exclusive, `std::shared_ptr` for shared, `winrt::com_ptr` for DX/COM
- **Diagnostics**: `DebugTrace(fmt, ...)` (debug-only via `OutputDebugString`), `DEBUG_ASSERT(expr)` for invariants (see `NeuronCore/Debug.h`)
- **Fatal errors**: `Neuron::Fatal(fmt, ...)` triggers `__debugbreak()` then throws `std::exception`
- **HRESULT handling**: mark functions `noexcept`, use `winrt::check_hresult`, allocate with `std::nothrow`
- **Enum iteration**: use `ENUM_HELPER(T, Start, End)` macro from `NeuronHelper.h` for range-for support

## Patterns to Avoid
- Raw owning pointers (use smart pointers)
- Macros for constants—prefer `constexpr`
- Implementation in headers (except templates)
- `using namespace` in headers

## Key Files to Read First
| File | Why |
|------|-----|
| `InterstellarOutpost/main.cpp` | Game loop, timing, input handling |
| `InterstellarOutpost/app.h` | `App` class – central coordinator |
| `NeuronCore/Debug.h` | `DebugTrace`, `DEBUG_ASSERT`, `Fatal` |
| `NeuronCore/NeuronCore.h` | Master PCH, WinRT imports, STL includes |
| `GameLogic/entity.h`, `building.h` | Entity/building type hierarchies |
| `GameRenderer/` | Client render logic and rendering architecture |

## Build & Run
```
Open InterstellarOutpost.slnx → Build (Ctrl+Shift+B) → F5
```
Assets in `InterstellarOutpost/Assets/` are deployed to output alongside the executable.

## Adding New Entity/Building Types
1. Add enum value in `Entity::Type*` or `Building::Type*`
2. Create `.cpp/.h` in `GameLogic/` deriving from base class
3. Update factory method (`Entity::NewEntity` / `Building::NewBuilding`)
4. Wire into `Location` if needed

## Documentation Rules
- **Document only what exists**—cite file:line when referencing patterns
- Ask clarifying questions rather than speculate on intent
- Review each documented item against source code

## Code Review Focus
- Adherence to `.editorconfig` and C++ Core Guidelines
- RAII and smart-pointer correctness
- Exception safety (`noexcept` where appropriate)
- Test coverage for public interfaces

## Client Render Logic Migration
- Migrate client render logic from `GameLogic` into `GameRenderer` to maintain separation of concerns and improve modularity.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Zwaliebaba) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
