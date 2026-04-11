# Club Fungal - AI Agent Instructions

## Architecture Overview

- **Service-Oriented Design**: Core logic in ScriptableObject services inheriting from `GURUService` (e.g., `SnapshotService`, `UnitControllerService`).
- **Data Flow**: Services reference each other (e.g., `SnapshotService` uses `UnitControllerService.Controllers`). Persistence via `LocalData` (JSON) and Unity assets.
- **Component Hierarchy**: Units/builds parented under `RoomController` (UnitParent/BuildParent). Repositioning triggers auto-save via `RepositionableEditor`.

## Critical: Simulator Pattern

**NEVER put game logic in Editor windows. They are ONLY for simulation and visualization.**

- **Editor Windows**: Located in `Assets/*/Editor/`. Used for simulating and testing game systems (e.g., `NetworkRunWindow`, `ActivityDrawer`). They ONLY draw UI and call methods on game components.
- **Game Logic**: Lives in reusable components/classes that will work in the actual game (e.g., `ActivityInstance`, `ResourceUpdateComponent`, `UnitInstance`, `Inventory`).
- **DRY Principle**: Logic exists in ONE place only. If you need to repeat logic, extract it to a shared component/service. Duplication causes inconsistencies.
- **Rule**: If you need to implement logic in an editor file, STOP. Create or find the appropriate game component/service method instead. Editor files should ONLY call existing methods, never implement logic directly.

Example: `NetworkRunWindow` simulates network runs but contains NO game logic. It calls `activity.Update(currentRun)` on `ActivityInstance` objects, which contain the actual logic.

Bad Example:

```csharp
// ❌ WRONG - Logic in editor file
if (GUILayout.Button("Claim Items"))
{
    unit.Inventory.RemoveItem(item, count);
    for (int i = 0; i < count; i++)
    {
        globalInventory.AddItem(item);
    }
}
```

Good Example:

```csharp
// ✅ CORRECT - Editor calls game component method
if (GUILayout.Button("Claim Items"))
{
    unit.ClaimItemsToGlobalInventory(item, count, globalInventory);
}
```

## Critical: Fix Root Causes, Not Symptoms

**NEVER add safeguards or workarounds. Always debug and fix the actual problem.**

- **No Band-Aids**: If something breaks after recompilation/serialization, fix the serialization issue (e.g., implement `ISerializationCallbackReceiver`), don't add checks to restore state.
- **Debug First**: Spend time understanding WHY something fails. Root cause fixes prevent future issues and work everywhere.
- **Safeguards Hide Problems**: Temporary fixes create tech debt and often fail in unexpected scenarios.

Example: When `UnitInstance.Skills` dictionary was lost after recompilation, the correct fix was implementing `ISerializationCallbackReceiver` to rebuild it from the serialized list, NOT adding `EnsureUnitHasSkills()` checks everywhere.

## Key Patterns

- **Asset Loading**: Use `GURUStyler.LoadAsset<T>(typeName)` for ScriptableObjects (e.g., `GURUStyler.LoadAsset<SnapshotService>("SnapshotService")`). Assumes assets exist; no null checks.
- **Movement**: Call `controller.Teleport(position, parent)` for precise positioning (updates transform and parent).
- **Snapshots**: `SnapshotInstance` assets store unit positions. Load/save via `SnapshotService` methods.
- **Editor UI**: Extend `GURUEditor` with `DrawContent()`. Use `GURUStyler.DrawGuruSection()` for styled sections with service buttons.
- **Inventory**: Use `Inventory.IsFull` to check capacity limits. `MaxCapacity` of 0 means unlimited. `TotalItemCount` returns sum of all items.
- **Simulation Speed**: `NetworkRunSettings.speedMultiplier` controls update speed (1.0 = normal, 2.0 = 2x speed, etc.). Applied to activity component timing calculations.

## Workflows

- **Build**: Unity iOS build via Xcode (see `Builds/` folder). Use `process_symbols.sh` for symbols.
- **Debugging**: Editor scripts auto-reposition units. Check `Logs/` for Unity output.
- **Persistence**: JSON in `LocalData.JsonFile`. Assets for snapshots. Auto-save on quit/scene unload.

## Conventions

- **Naming**: Services end with "Service", editors with "Editor". Use `GURU` prefix for custom classes.
- **Initialization**: Services initialize in `OnInitialize()` or `OnSceneLoaded()`.
- **Error Handling**: Minimal; assume valid state. Let code error naturally for easier debugging—don't add null checks, warnings, or safeguards. If something is missing, a null reference exception at the exact usage line is clearer than a generic warning.
- **Coding Style**: Write code without null checks; assume all dependencies are available.

## Examples

- Load service: `var service = GURUStyler.LoadAsset<SnapshotService>("SnapshotService"); service.SaveSnapshot();`
- Teleport unit: `controller.Teleport(newPos, room.UnitParent);`
- Asset selector: Use `AssetSelectorComponent<T>` for drag-drop with history.

### Simulator Pattern in Practice

**✅ CORRECT**: Editor window draws, game component has logic

```csharp
// In Editor window
private void UpdateLoop() {
    activity.Update(currentRun); // Logic in ActivityInstance
    Repaint(); // Editor just redraws
}
```

**❌ WRONG**: Logic duplicated in editor window

```csharp
// Don't put game logic here - it won't work in the actual game!
private void UpdateLoop() {
    currentRun.Inventory.AddItem(item); // Logic shouldn't be here
}
```

**✅ CORRECT**: When you need skills in copied units

```csharp
// Add logic to UnitInstance.Copy() method (game logic)
// Editor window just calls: var copy = original.Copy();
```

### Unit Skills Pattern

When copying UnitInstance objects, skills must be preserved:

- `UnitInstance.Skills` is a `Dictionary<Skill, SkillInstance>` (use `.Values` to iterate)
- Copy skills by XP: `new SkillInstance(unitCopy, skill, originalSkillInstance.XP)`
- Dance skill is special: `new DanceSkillInstance(unit, skill, xp)` for `skill.Id == "dance"`
- Auto-fill missing skills from `skillCollection` during initialization (see `UnitInstanceService.OnInitialize()`)

Reference: `Assets/Snapshots/`, `Assets/Editor/`, `Assets/Units/Scripts/`, `Assets/NetworkRun/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jmonzone)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/jmonzone)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
