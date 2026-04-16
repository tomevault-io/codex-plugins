## MySnippets Plugin — Copilot Instructions

These notes are targeted at AI coding agents editing this repository. Keep guidance concise and concrete — reference files shown below when making changes.

- Purpose: Desktop-only Obsidian plugin that surfaces CSS snippets in a status-bar menu and provides quick actions (reload, open folder, create snippet).

Key files

- `src/plugin/main.ts`: plugin entry point. Lifecycle hooks used: `onload()` and `onunload()`. `setupSnippetsStatusBarIcon()` registers the status-bar button, commands, and click handlers that call into the UI module.
- `src/ui/snippetsMenu.ts`: builds the status-bar menu using Obsidian `Menu`, `ToggleComponent`, `ButtonComponent`. Reads `app.customCss` for `snippets`, `enabledSnippets`, and folder/path helpers.
- `src/modal/createSnippetModal.ts`: modal that creates new snippet files via `app.vault.create(...)` and interacts with `app.customCss` to request a reload and toggle status.
- `src/settings/*`: settings model (`settingsData.ts`), settings UI (`settingsTab.ts`), and helper types (`type.ts`) defining `EnhancedApp`/`EnhancedMenu` partial typings used across the codebase.
- `src/icons/customIcons.ts`: single source of SVG icons; `addIcons()` is invoked in `onload()`.
- `src/util/setAttributes.ts`: small DOM helper used throughout for attribute setting.
- `manifest.json`: plugin metadata; note `isDesktopOnly: true` (affects compatibility).
- `package.json`: dev scripts — `npm run dev` (rollup watch) and `npm run build` (production build).

Architecture and data flow (big picture)

- Obsidian plugin model: `MySnippetsPlugin` (class in `src/plugin/main.ts`) is instantiated by Obsidian. `onload()` sets up icons, loads settings, registers a Setting tab and, after layout is ready, creates a status-bar item.
- UI is lightweight and mostly delegated to `snippetsMenu()` which reads the current snippet list from `app.customCss` and attaches controls (toggles, buttons). Actions call Obsidian APIs (e.g., `openWithDefaultApp`, `vault.create`) or `customCss` helper methods (e.g., `setCssEnabledStatus`, `requestLoadSnippets`).
- Settings state flows: `loadSettings()` uses `this.loadData()` merged with `DEFAULT_SETTINGS` in `src/settings/settingsData.ts`. `saveSettings()` calls `this.saveData()`.

Project-specific patterns & conventions

- Absolute-style imports: code uses `import ... from "src/.."` (not relative `../`). Assume TypeScript/rollup is configured for `baseUrl` or path resolution. Avoid changing import style unless updating build config.
- Enhanced types: `src/settings/type.ts` defines lightweight, partial typed interfaces for the Obsidian `App`, `Menu`, and `MenuItem` objects (e.g., `EnhancedApp.customCss`). When modifying behavior toward Obsidian internals, update these types to reflect the surface used by the plugin.
- Single icon registry: all custom SVGs are in `src/icons/customIcons.ts`. To add or adjust icons, update this file and ensure `addIcons()` remains called from `onload()`.
- UI uses Obsidian components directly rather than external UI libs. Keep using `Menu`, `Modal`, `Setting`, `ToggleComponent`, `ButtonComponent` for consistency.
- Settings persistence: settings are stored using Obsidian's `loadData()`/`saveData()` pattern. Keep `DEFAULT_SETTINGS` shape in sync with `MySnippetsSettings` interface.

Build & developer workflow (concrete commands)

- Install dev deps: `npm install` (standard). The project is TypeScript + Rollup.
- Development (watch): `npm run dev` — runs `rollup --config rollup.config.js -w` and will output `main.js` (see `rollup.config.js`). Use this while editing to produce build artifacts to load into Obsidian.
- Production build: `npm run build` — runs rollup in production mode. After build, copy the output files into your Obsidian vault plugin folder or use your usual Obsidian plugin deploy workflow.
- Debugging in Obsidian: add `console.log()` statements in `onload()` / handlers (this repo already uses console logs). Rebuild, then reload plugin within Obsidian to see logs in Developer Console.

What to look for when editing

- If changing any API usage of `app.customCss` or `app.vault`, update `EnhancedApp` in `src/settings/type.ts` to document those calls.
- When adding new commands or status-bar behavior, mirror the pattern in `setupSnippetsStatusBarIcon()` in `src/plugin/main.ts` (addStatusBarItem, addClass on element, setIcon, click listener, and this.addCommand()).
- When updating the menu UI, prefer `snippetsMenu()`; it expects `settings` as a plain POJO of `MySnippetsSettings` from `settingsData.ts`.
- Keep DOM style adjustments localized in the UI files (`snippetsMenu.ts` or `createSnippetModal.ts`) and use `setAttributes()` utility for attribute changes.

Safety and compatibility notes

- `manifest.json` lists `minAppVersion` and `isDesktopOnly: true`. Avoid introducing mobile-only APIs or web-only expectations.
- The plugin relies on Obsidian internals like `customCss` object; these are not standard JS objects — treat them as opaque helpers and update typings when expanding usage.

Quick examples (copyable)

- Add a status-bar command (pattern from `main.ts`):
  - See `setupSnippetsStatusBarIcon()` — it registers `this.addCommand({ id, name, icon, callback })` and uses `this.addStatusBarItem()` to create clickable UI.
- Read snippet list and toggle status (pattern from `snippetsMenu.ts`):
  - `const currentSnippets = app.customCss.snippets;`
  - `customCss.setCssEnabledStatus(snippet, !isEnabled)`

Merge guidance

- If a `.github/copilot-instructions.md` already exists, preserve any unique human-written guidance and merge the concrete items above. This file is intentionally short — prefer small, code-referenced bullets instead of broad best-practices.

If anything here is unclear or you want me to expand a specific section (deploy steps, rollup output locations, or type definitions), tell me which area to expand and I'll iterate.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/RichLewis007) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
