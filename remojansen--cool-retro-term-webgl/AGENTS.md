# Copilot Instructions for cool-retro-term-webgl

## Architecture Overview

This is a **npm workspaces monorepo** containing two packages:

- **`cool-retro-term-renderer`** - WebGL CRT effects library using Three.js and XTerm.js
- **`cool-retro-term-electron`** - Electron desktop app using node-pty for real shell integration

The renderer follows a **two-pass GLSL shader pipeline** ported from the original QML-based [cool-retro-term](https://github.com/Swordfish90/cool-retro-term):
1. **Static Pass**: Screen curvature, RGB shift, bloom, brightness → renders to `staticRenderTarget`
2. **Dynamic Pass**: Flickering, burn-in, jitter, rasterization → reads from static pass, renders to screen

Burn-in uses **ping-pong buffers** for frame accumulation (see `burnInRenderTargets` in [TerminalText.ts](modules/cool-retro-term-renderer/src/TerminalText.ts#L255-L285)).

## Key Commands

```bash
npm install                  # Install all workspace dependencies
npm run build                # Build both packages
npm run start:electron       # Run the Electron app

# After modifying native dependencies (node-pty):
cd modules/cool-retro-term-electron && npx @electron/rebuild
```

**Critical**: `node-pty` must be rebuilt for Electron's Node.js version, not system Node. Always run `@electron/rebuild` after `npm install` or when switching Electron versions.

## Code Conventions

- **Linting/Formatting**: Biome with tabs, double quotes. Run `npm run lint-and-format`
- **Shader code**: GLSL strings use `/* glsl */` template tag for syntax highlighting
- **Effect modules**: Each CRT effect is a standalone GLSL module in [shaders/effects/](modules/cool-retro-term-renderer/src/shaders/effects/) exporting a `*GLSL` constant

## Electron IPC Pattern

The Electron app uses **contextBridge** for secure IPC:
- Main process ([main.ts](modules/cool-retro-term-electron/src/main.ts)): Manages `node-pty` process, handles `terminal:*` IPC events
- Preload ([preload.ts](modules/cool-retro-term-electron/src/preload.ts)): Exposes `window.terminalAPI` via contextBridge
- Renderer ([renderer.ts](modules/cool-retro-term-electron/src/renderer.ts)): Uses `window.terminalAPI` for PTY communication

Data flow: Keyboard → XTerm.js → IPC → node-pty → shell → node-pty → IPC → XTerm.js → CRT renderer

## Adding New CRT Effects

1. Create GLSL module in `modules/cool-retro-term-renderer/src/shaders/effects/`
2. Export the GLSL string constant (e.g., `export const myEffectGLSL = ...`)
3. Add to [effects/index.ts](modules/cool-retro-term-renderer/src/shaders/effects/index.ts)
4. Add uniform in `TerminalText` constructor and create setter method
5. Integrate into static or dynamic pass fragment shader based on whether effect is time-dependent

## XTerm Integration

XTerm.js is rendered **off-screen** (positioned at `-9999px`) and used only for buffer/input management. The visible output goes through:
- `XTermConnector` syncs XTerm buffer → `TerminalText.setText()`
- `TerminalText` renders to canvas → Three.js texture → CRT shader pipeline

Grid size is calculated from window dimensions and font metrics in `calculateFontMetrics()`. Changes trigger `onGridSizeChange` callback to resize XTerm.

## Assets

- **Font**: Terminus font embedded as base64 in [font.ts](modules/cool-retro-term-renderer/src/assets/font.ts)
- **Noise texture**: Base64 in [noise.ts](modules/cool-retro-term-renderer/src/assets/noise.ts)
- Regenerate with: `cd modules/cool-retro-term-renderer && npm run generate-assets`

---
> Source: [remojansen/cool-retro-term-webgl](https://github.com/remojansen/cool-retro-term-webgl) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-03 -->
