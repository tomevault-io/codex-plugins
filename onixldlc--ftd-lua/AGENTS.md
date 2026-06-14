# Copilot Instructions — FTD-lua

## What This Is
Lua scripts for **From The Depths** (FTD), a game with a Lua scripting API. Scripts run inside an in-game editor with **no debugger, no `require()`, no undo** — broken code is painful to fix. **When unsure about an API call, ask the user or leave a `-- TODO:` rather than guessing.**

## API Reference
- **Full docs:** `docs/ftd-api-reference.md` — use for parameter details, data type fields, and edge cases
- **Quick lookup:** `docs/ftd-api-quick-reference.md` — compact method listing for fast context

## FTD Lua Basics
- Entry point is always `function Update(I)` — called **40 times/second**
- `I` is the game interface; all API calls use `I:Method()` (colon syntax)
- Libraries: `Mathf.Func()` (dot syntax), `Vector3(x,y,z)`, `Quaternion` — Unity APIs exposed by FTD
- No module system — every script must be **fully self-contained** (no `require()`)
- **Code is pasted directly into the in-game editor** — there is no file save, no undo, no version control inside the game. If a helper/library is needed, it must be **inlined into the single script file**. Getting code wrong means manually retyping or re-pasting everything.
- **NEVER hallucinate API calls.** If unsure whether a function exists or what it returns, ask the user or leave a `-- TODO:`. A wrong function call fails silently in-game and is extremely painful to debug.

## Project Structure
```
missiles/    — Lua-guided missile scripts (AA, kinetic strike, supersonic)
failed/      — Scripts that failed due to game engine limitations (radar accuracy, targeting API restrictions) — kept as a graveyard, NOT as code reference
util/        — Template, diagnostics, and reusable utility scripts
other/       — Data lookup tables (component IDs, weapon type enums)
docs/        — API reference documentation
```

## Codebase Conventions
- **Use the template pattern** from `util/script-template.lua`: separate `init()` (once), `every_tick()` (40Hz), `every_second()` (1Hz), with `update_deltaT()` for timing
- **Start `Update()` with `I:ClearLogs()`** — the log only holds 100 messages
- **Cache `I` as global `Game`** for use in helper functions outside `Update()`:
  ```lua
  Game = nil
  function Update(I)
      if Game == nil then Game = I end
  end
  ```
- **Variables are globals** (no `local` at top level) — this is intentional for persistence across frames
- **snake_case** for variables and functions (`target_pos`, `get_target_list`)
- **Section delimiters** for organizing longer scripts:
  ```lua
  -- ===============LOGICS STARTS===============
  -- ================LOGICS ENDS================
  ```

## Common Patterns

**Target acquisition** — mainframe index must be a **configurable global variable** (a craft may have multiple mainframes for different roles, e.g. one for radar/lua, one for guns/AI):
```lua
mainframe = 0  -- global, set per script
target = I:GetTargetInfo(mainframe, 0)
target_pos = target.Position
target_vel = target.Velocity
```

**Missile guidance loop** — always nested transceiver → missile:
```lua
for t = 0, I:GetLuaTransceiverCount() - 1 do
    for m = 0, I:GetLuaControlledMissileCount(t) - 1 do
        missile = I:GetLuaControlledMissileInfo(t, m)
        I:SetLuaControlledMissileAimPoint(t, m, x, y, z)
    end
end
```

**Velocity from position deltas** — the project derives velocity manually rather than trusting `target.Velocity`, using a rolling average buffer (see `missiles/missile-AA.lua`).

## Key Gotchas
- FTD Lua indices are **0-based** (unlike standard Lua which is 1-based)
- `Vector3` supports operators (`+`, `-`, `*`, `.magnitude`) — prefer these over custom `vecDiff` helpers
- `I:SetLuaControlledMissileAimPoint` takes **global world coordinates**, not direction vectors
- `DetonateLuaControlledMissile` and `SetLuaControlledMissileAimPoint` require a **Lua receiver ON the missile** to work
- There is no shared state between scripts on different constructs
- `dt` is `1/40` seconds per tick (use `I:GetTime()` for actual elapsed time)
- **FTD arrays (LuaArray) are C# userdata, NOT Lua tables.** `ipairs()`/`pairs()` will error. `.Count` does not exist. Use `#array` for length and numeric `for` loops to iterate. Wrap in `pcall` if unsure about 0-based vs 1-based indexing.
- **`WeaponInfo` confirmed fields:** `Valid`, `LocalPosition`, `GlobalPosition`, `LocalFirePoint`, `GlobalFirePoint`, `Speed`, `CurrentDirection`, `WeaponType`, `PlayerCurrentlyControllingIt`. **`WeaponSlot` and `WeaponSlotMask` do NOT exist at runtime** despite appearing in some documentation.

---
> Source: [onixldlc/FTD-lua](https://github.com/onixldlc/FTD-lua) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-14 -->
