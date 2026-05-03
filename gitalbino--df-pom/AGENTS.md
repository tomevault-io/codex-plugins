# Copilot Instructions - DFPOM (Dwarf Fortress Production Orders Manager)

## Project Overview
An Electron desktop app that provides an improved UI to manage Dwarf Fortress "Fortress Mode" production orders. It communicates with a running Dwarf Fortress game via DFHack's `dfhack-run.exe` and Lua scripts.

## Architecture

### Core Data Flow
1. **Lua scripts** (`gameInfo.lua`, `exportStocks.lua`, `jobInfos.lua`) extract data from DF via DFHack
2. **Data passes via clipboard** - Lua writes JSON to clipboard, Electron reads it
3. **main.js** (main process) executes `dfhack-run.exe` with Lua scripts, handles IPC
4. **preload.js** exposes IPC API to renderer via `window.api`
5. **df-pom.js** (renderer) manages UI state, orders manipulation, inventory display

### Key Files
- [main.js](main.js) - Electron main process: config management, DFHack communication, file I/O
- [df-pom.js](df-pom.js) - Renderer logic: UI state, orders table, stocks inventory, event handlers
- [df-pom-ordersdata.js](df-pom-ordersdata.js) - Order property definitions and metadata
- [df-pom-smeltingOrders.js](df-pom-smeltingOrders.js) - Pre-configured smelting order templates
- `*.lua` files - DFHack scripts for extracting game data (stocks, jobs, materials)

### DFHack Communication Pattern
```javascript
// main.js executes Lua via dfhack-run.exe, Lua writes result to clipboard
execFile(path, ["lua", "-f", luaScriptPath], (error, stdout, stderr) => {
	let data = clipboard.readText();
	// Parse JSON, fix trailing commas: data.replace(/(,)+}/g, "}")
});
```

## Code Conventions

### Indentation & Formatting
- Use **tabs**, not spaces
- Private variables: prefix with `m_` using camelCase (e.g., `m_myVariable`)
- Bracket style for `if` statements:
```javascript
// Single instruction - no brackets
if (condition)
	doSomething();

// Multiple instructions - brackets on next line
if (condition)
{
	doSomething();
	doSomethingElse();
}
```

### DOM Querying
The app uses a shorthand selector: `const $ = (selector) => document.querySelectorAll(selector);`

### Async Patterns
- Queue-based operations: `QueueOrdersRead()`, `QueueOrdersSave()`, `QueueStocksRead()`
- Auto-update loop in `DataAutoUpdater()` runs every 150ms
- Use `DELAY_BETWEEN_FILE_OPS_MS` (3500ms) between file operations

### State Management
- Global variables track UI state (e.g., `orders`, `stocks`, `gm` for game materials)
- Order changes tracked via `MarkEdited(order)` → DOM class `edited`
- Pause states use constants: `PAUSECHANNEL_ALLSTASKS`, `PAUSECHANNEL_FROMTASK`, etc.

## Running the App
```powershell
npm install
npm start
```
Requires Dwarf Fortress with DFHack installed. App prompts for paths on first run.

## Lua Script Template Pattern
Template scripts use placeholder values replaced at runtime:
```lua
-- In template (exportStocks.lua):
local startIndex = 69420;  -- placeholder

-- In main.js:
luaScriptContent = luaScriptContent.replace("69420;", stocksReaderStartIndex + ";");
```

## IPC Handlers (main.js ↔ renderer)
Key handlers exposed via `window.api`:
- `ReadOrdersFile` / `WriteOrdersFile` - Orders JSON via DFHack
- `GetGameInfos` - Materials, items, job types from DF
- `GetStocks` - Current fortress inventory
- `GetJobsInfos` - Available job definitions
- `GetGameStatus` - Check if DF is running with a fortress loaded

## DFHack Quirks & Naming Inconsistencies
DFHack has irregular naming conventions that require bruteforce handling:
- Item types sometimes need a trailing `S`, sometimes not (e.g., `BAR` vs `BARS`)
- Job descriptions are often not directly available from the API
- Many mappings between game IDs and human-readable names are hardcoded
- Expect `if/then` chains for special cases - this is intentional, not tech debt

## Testing
Testing requires Dwarf Fortress running with DFHack and a loaded fortress. No automated tests - manual verification against live game data.

---
> Source: [GitAlbino/df-pom](https://github.com/GitAlbino/df-pom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-02 -->
