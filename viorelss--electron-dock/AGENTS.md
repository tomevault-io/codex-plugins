# Copilot Instructions for DockApp

## Project Overview
- Electron desktop app that hosts an Angular 19 frontend bundled to `dist/dock-app`.
- Main Electron process entry: `main.js` (tray icon, window management, IPC handlers).
- Angular browser entry: `src/main.ts` bootstrapping standalone `AppComponent` with router.
- Packaging and distribution are handled by Electron Forge (`forge.config.js`, `electron-forge` scripts in `package.json`).

## Architecture & Data Flow
- **Main window**: created in `createWindow` in `main.js`, loads `dist/dock-app/browser/index.html` and uses `preload.js` for a safe bridge.
- **Preload bridge**: `preload.js` exposes a `window.electron` API via `contextBridge` with:
  - `ipcRenderer.send(channel, data)` / `on(channel, listener)` / `invoke(channel, data)` wrappers.
  - `getPathForFile(file)` using `webUtils.getPathForFile`, emitting a `"file-path"` IPC event.
  - `getAppIcon(filePath, isDirectory)` calling the `"get-app-icon"` IPC handler.
- **Angular integration**:
  - Global typings are in `src/electron.d.ts` (ElectronAPI and `window.electron`). When extending the preload bridge, update this file to keep TS happy.
  - Root component `AppComponent` (src/app/app.component.ts) renders the router outlet plus the dock (`DockComponent`). Routing is currently empty (`app.routes.ts`).
- **Dock behavior** (`DockComponent` in src/app/dock/dock.component.ts):
  - Listens to `"mouse-position"` IPC events to toggle visibility when the cursor is near the top-center of the active display.
  - On initialization, loads apps via `ipcRenderer.invoke("read-apps")` and listens for `"apps-cleared"` to reset state.
  - Supports drag-and-drop of files: uses `window.electron.getPathForFile` and `getAppIcon`, then persists apps via `"write-apps"` IPC.
  - Launches apps via `"launch-app"` or opens folders via `"open-folder"` using `ipcRenderer.send`.
- **Persistence & icons** (main process):
  - Apps are stored in `apps.json` (JSON array) alongside `main.js` via `read-apps` / `write-apps` IPC handlers.
  - Icons are resolved in `getIconPathFromShortcut` and `ipcMain.handle("get-app-icon", ...)` using `windows-shortcuts`, `nativeImage`, and fallback assets in `public/`.

## Workflows & Commands
- **Angular dev server only** (no Electron shell):
  - `ng serve` (see README.md) – runs the web app at `http://localhost:4200`.
- **Electron + Angular together**:
  - `npm run electron` – builds Angular with `ng build` then starts Electron against the built output.
  - `npm start` – `electron-forge start` (preferred for iterating on the Electron app).
- **Build & packaging**:
  - `npm run build` – Angular production build to `dist/dock-app` (used by Electron).
  - `npm run package` / `npm run make` – Electron Forge packaging and distributables, configured in `forge.config.js`.
- **Tests**:
  - Frontend unit tests use Karma/Jasmine (`ng test`). Test files are currently disabled by schematics defaults; enable tests per-component when adding new ones.

## Conventions & Patterns
- **Angular style**:
  - Standalone components (no NgModule); imports declared directly on `@Component({ imports: [...] })`.
  - Default styling is SCSS; Tailwind is configured via `tailwind.config.js` and `src/styles.scss`.
  - Skip tests by default for new schematics (see `angular.json`); only add tests where they provide clear value.
- **Electron IPC patterns**:
  - IPC channels are plain strings (e.g., `"mouse-position"`, `"read-apps"`, `"write-apps"`, `"get-app-icon"`, `"launch-app"`, `"open-folder"`).
  - For **new IPC channels**:
    1. Add handlers in `main.js` using `ipcMain.on` or `ipcMain.handle`.
    2. Add a wrapper in `preload.js` under the `contextBridge.exposeInMainWorld` object.
    3. Update `ElectronAPI` in `src/electron.d.ts` so Angular code gets type safety.
    4. Consume via `window.electron` in Angular, preferably wrapped in a service when used in multiple places.
- **Change detection & zones**:
  - `DockComponent` uses `NgZone` and `ChangeDetectorRef` to run IPC callbacks inside Angular’s zone and explicitly trigger `detectChanges()`. Follow this pattern for any IPC-driven UI updates to avoid change detection errors.
- **File and icon handling**:
  - Use `webUtils.getPathForFile` (already wrapped as `getPathForFile`) for resolving OS file paths from drag-and-drop events.
  - Treat directories vs executables differently in the main process and UI (see `isDirectory` / `isExecutable` checks in `DockComponent` and `main.js`).

## Tips for Making Changes
- When modifying dock behavior (visibility rules, max apps, drag/drop), keep logic centralized in `DockComponent` and reuse existing IPC channels where possible.
- When changing the build output path or Angular project name, ensure Electron’s `loadFile` path in `main.js` and Forge config still point at the correct `dist/.../browser/index.html`.
- For new UI features that depend on system state, prefer main-process logic with a thin preload/Angular bridge rather than enabling `nodeIntegration` in the renderer.

## Where to Look First
- Main process behavior: `main.js`
- Renderer bootstrap & configuration: `src/main.ts`, `src/app/app.config.ts`, `src/app/app.component.*`
- Dock UI and interactions: `src/app/dock/dock.component.*`
- Electron bridge & typings: `preload.js`, `src/electron.d.ts`
- Build and packaging: `package.json`, `forge.config.js`, `angular.json`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ViorelsS) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
