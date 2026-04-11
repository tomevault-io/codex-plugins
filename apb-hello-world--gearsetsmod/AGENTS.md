# Copilot Review Instructions — GearSetsMod

## Project overview

GearSetsMod is a BepInEx plugin for **Tainted Grail: Fall of Avalon** that adds a "Sets" tab to the character sheet, letting players save and load gear, talent, and attribute loadouts. It can pull items from the stash and return them when switching sets.

The mod has no access to game source code — it relies heavily on **reflection** and **Harmony patches** to interact with game internals.

## Architecture

```
Plugin.cs                    — Entry point, Harmony bootstrap
Core/
  GearSet.cs                 — Data model (serialized to JSON)
  SetManager.cs              — File-based persistence (save/load/delete)
  GearSetCapture.cs          — Captures current hero state into a GearSet
  GearSetApply.cs            — Applies a saved GearSet to the hero
  SlotHelpers.cs             — Slot/enum name resolution, item name lookup
  JsonWrappers.cs            — JSON serialization abstraction
UI/
  GearSetsUI.cs              — Controller (click handlers, set list, detail panel)
  VGearSetsUI.cs             — View layout builder (Unity UI construction)
  NameInputDialog.cs         — IMGUI modal dialog for save/delete confirmation
Patches/
  GearSetsTabPatch.cs        — Harmony patches to inject the Sets tab button
```

## Key review concerns

### Reflection safety
Most interaction with game types uses reflection. Watch for:
- Missing null checks on `GetProperty`, `GetField`, `GetMethod` results
- Incorrect `BindingFlags` (e.g. forgetting `NonPublic` or `FlattenHierarchy`)
- Unchecked casts from `GetValue` — the return type may change between game updates

### Stat handling (critical bug area)
The stat system was the source of all 4 user-reported bugs. The rules are:
- **Always read `BaseValue`**, never `Modified`/`ModifiedValue` (those include gear/talent bonuses)
- **Always use `Stat.SetTo(value, false, null)`** to write stats — never `IncreaseBy`/`DecreaseBy`
- **Never call `HeroRPGStats.RecalculateAllStats()`** — it resets stats from an internal wrapper
- Instead call `hero.HeroStats.RecalculateAllStats(false)` and `hero.CharacterStats.RecalculateAllStats(level, level, false)`

### Stash integration
- `HeroStorage.RequestItems()` must be called before accessing `Items`, and `ReleaseItems()` must be called when done (ref-counted)
- Only items explicitly pulled from stash should be tracked in `_lastStashPulledGuids` — don't return arbitrary unequipped items
- `item.MoveTo(target)` can return null if the target is full

### v1 migration
Sets saved before v2 stored `Modified` values instead of `BaseValue`. The `MigrateV1StatValue` method approximates BaseValue by subtracting current modifiers. Any change to this logic should be carefully reviewed.

### Build system
- The `.csproj` uses `EnableDefaultCompileItems=false` — new `.cs` files **must** be added as `<Compile Include="..."/>` entries or they won't compile
- Game DLLs in `lib/` are proprietary and must never be committed
- The GitHub release workflow packages a pre-built DLL from `dist/`, not from CI build output

## Code style

- Use `LogDebug` for routine tracing, `LogInfo` only for significant lifecycle events (plugin loaded, tab injected)
- Use `LogWarning` for recoverable issues, `LogError` for failures that break functionality
- Never use empty `catch { }` blocks — at minimum log or comment why the exception is intentionally swallowed
- Wrap reflection-heavy code in try/catch with descriptive log messages — game updates can break reflection targets silently
- Keep methods under ~40 lines where practical; extract named helpers for distinct logical steps

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/apb-hello-world)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/apb-hello-world)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
