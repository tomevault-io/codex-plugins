# Modbus Client Project Instructions

## Tech Stack & Architecture
- **Framework**: Quasar v2 (Vue 3, Vite) wrapping Electron v33.
- **Language**: TypeScript used in both Main (`src-electron`) and Renderer (`src`) processes.
- **State Management**: Pinia (`src/stores/`).
- **Communication**: Strict IPC via `contextBridge`.
  - **Main**: `src-electron/electron-main.ts` handles hardware (`modbus-serial`).
  - **Preload**: `src-electron/electron-preload.ts` creates the bridge.
  - **Renderer**: Uses `window.myAPI` defined in `src/my-api.d.ts`.

## Core Workflows
1.  **Running the App**:
    - Use `npm run dev` (runs `quasar dev -m electron`).
    - Note: This compiles Main/Preload explicitly. Watch out for linting errors in Main process blocking the build.
2.  **Modifying Modbus Logic**:
    - **Step 1 (Main)**: Add `ipcMain.handle` in `electron-main.ts` using `modbus-serial` client.
    - **Step 2 (Preload)**: Expose method in `electron-preload.ts` via `ipcRenderer.invoke`.
    - **Step 3 (Types)**: Update `src/my-api.d.ts` to extend `Window` interface.
    - **Step 4 (Store)**: Call `window.myAPI.method()` in `src/stores/modbus-store.ts`.

## Project Conventions
- **UI Components**: Exclusively use Quasar components (`q-btn`, `q-input`, `q-select`, `q-card`).
  - Prefer `dense` and `outlined` props for inputs.
  - Use `q-btn-toggle` for mode switching.
- **Type Safety**:
  - Avoid `any` in `electron-main.ts` and `electron-preload.ts`. Use union types (e.g., `string | number | boolean`) for incoming IPC args.
  - Do NOT use `any` in try/catch blocks (`catch (e: any)`). Use `unknown` or handle safely with type checks (`instanceof Error`).
  - Ensure `my-api.d.ts` matches `electron-preload.ts` exactly.
- **Reactive State**:
  - Store connection state and logs in Pinia (`modbus-store.ts`).
  - Components should read from Store via computed properties or directly.

## Key Files
- `src-electron/electron-main.ts`: Backend logic, Modbus connection management.
- `src-electron/electron-preload.ts`: Safe API exposure.
- `src/my-api.d.ts`: Typings for the IPC bridge.
- `src/stores/modbus-store.ts`: Central business logic and API consumer.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/filipek92)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/filipek92)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
