# BMAD STUDIO - AI Coding Instructions

## Architecture Overview
- **Tech Stack**: [React 19](package.json), [Tauri 2.0](src-tauri/tauri.conf.json), [TypeScript](tsconfig.json), [Vite](vite.config.ts), [Tailwind CSS 4](package.json).
- **Frontend-Backend Bridge**: Uses [Tauri commands](src-tauri/src/lib.rs) for system-level operations (Git, File System, Keyring).
- **Core Domain**: A studio for managing the "BMAD" framework.
    - `_bmad/`: Contains agent definitions, workflows, and templates (The Brain).
    - `_bmad-output/`: Contains generated artifacts from agent executions (The Workshop).

## Key Components & Libraries
- **Graph UI**: Powered by [@xyflow/react](src/components/FlowCanvas.tsx). Custom nodes are in [src/components/BmadNode.tsx](src/components/BmadNode.tsx).
- **Editors**:
    - [MarkdownEditor.tsx](src/components/MarkdownEditor.tsx) uses Milkdown and Tiptap.
- **AI Integration**:
    - [AIChatPanel.tsx](src/components/AIChatPanel.tsx) manages conversations with agents.
    - [BmadBrainPanel.tsx](src/components/BmadBrainPanel.tsx) visualizes/manages agent configuration.
- **UI & Icons**:
    - [lucide-react](package.json) for all iconography.
    - [framer-motion](package.json) for smooth transitions and animations.
    - [tailwind-merge](package.json) and `clsx` for dynamic class management.

## Developer Workflows
- **Running the App**: 
  - `npm run dev`: Starts the Vite development server (frontend only).
  - `npm run tauri dev`: Starts the full Tauri application (recommended for testing FS/Git features).
- **Adding Commands**: 
  - Define in [src-tauri/src/lib.rs](src-tauri/src/lib.rs) with `#[tauri::command]`.
  - Export in the `tauri::Builder` in `run()`.
  - Invoke from frontend using `invoke("command_name", { args })`.

## Conventions & Patterns
- **Styling**: Tailwind CSS 4 utility classes. Currently using template literals and string concatenation for dynamic classes.
- **State Management**: Primarily React `useState` and `useEffect` at the [App.tsx](src/App.tsx) level for global state.
- **Secrets**: NEVER hardcode API keys. Use `invoke("store_secret", ...)` to store keys in the system keychain via the backend.
- **File Access**: Use [@tauri-apps/plugin-fs](package.json) and [@tauri-apps/plugin-dialog](package.json) for user-triggered file operations.
- **Language**: The project configuration ([_bmad/bmm/config.yaml](_bmad/bmm/config.yaml)) specifies French as the default communication and document language.

## Project Structure
- `src/components/`: Core UI components.
- `src/assets/`: Static assets.
- `src-tauri/`: Rust backend and Tauri configuration.
- `_bmad/`: Configuration for the BMAD engine (agents, workflows).
- `_bmad-output/`: Generated documents and logs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MAW5-spec)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/MAW5-spec)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
