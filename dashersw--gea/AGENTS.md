## Purpose

These notes orient an AI coding agent to the `gea-tools` codebase so it can be productive immediately. The extension now targets the modern Gea JSX API, so focus on JSX-aware component discovery, LSP behavior, and the TypeScript plugin.

## Quick facts

- Main language: TypeScript (compiled to `out/`)
- Extension entrypoints: `out/client.js` (language client) and `out/server.js` (LSP server)
- Build: `npm install` then `npm run compile` (see `package.json` scripts)
- Activate in VS Code: press F5 (Extension Development Host) or `code --extensionDevelopmentPath=.`

## Key files to read first

- `package.json` — activation events, client/server entry, scripts, and TS server plugin contribution.
- `src/client.ts` — language client wiring and document selectors for JS/TS/JSX/TSX.
- `src/server.ts` — LSP diagnostics, completion, hover, and JSX tag context parsing.
- `src/component-discovery.ts` — heuristics for discovering class and function components plus their props.
- `ts-plugin/gea-template-plugin.js` — TypeScript server plugin that suppresses noisy JSX diagnostics and unused-import warnings for imported JSX components.
- `language-configuration.json` — still present, but no custom grammar is used anymore.

## Architecture / data flow (summary)

1. The client (`src/client.ts`) starts a language client that launches the server (`out/server.js`).
2. The server (`src/server.ts`) listens to open and changed documents, discovers local/imported components, then provides JSX-aware completions, hover info, and unknown-component diagnostics.
3. `ComponentDiscovery` extracts props from component signatures:
   - class components: `template({ ... })`
   - function components: `function Name({ ... })`
   - body destructuring: `const { ... } = props`
4. The TypeScript plugin runs inside tsserver to suppress noisy JSX diagnostics in Gea component bodies and to suppress TS6133 when imported components are only referenced as JSX tags.

## Project conventions and important patterns (do not break)

- Modern Gea components use JSX, not backtick template syntax.
- Class components usually extend `Component` and render from `template(props)`.
- Function components are plain functions returning JSX.
- Props should be inferred from parameter destructuring or `props` destructuring in the body.
- JSX event attributes use Gea's lowercase convention such as `click`, `input`, `change`, and `keydown`.
- `class` is valid in Gea JSX and should not be treated as a React mistake.
- Default-imported local components are the main discovery/completion path right now.

## Build & debug workflows (explicit)

1. Install deps: `npm install` in the extension root.
2. Compile: `npm run compile` (runs `tsc -p ./`, outputs JS into `out/`). Always re-run after TypeScript edits.
3. Run extension in VS Code: open the workspace and press F5. Client uses `out/client.js` and server `out/server.js`.
4. Debug server: `src/client.ts` sets `--inspect=6009` for debug mode. Use the Debug panel in the Extension Development Host to attach to that port if needed.
5. TypeScript plugin: it's registered via `contributes.typescriptServerPlugins` in `package.json` and loaded by the TS server inside the editor; logs go to the TypeScript output panel.

## Where to make changes for common tasks

- Add or change component discovery heuristics: edit `src/component-discovery.ts` and be careful with `file://` URI handling and import resolution.
- Add language features (completion, hover, diagnostics): modify `src/server.ts` and test in `.jsx` and `.tsx` files.
- Change TS plugin behavior: edit `ts-plugin/gea-template-plugin.js`; remember it runs inside tsserver and logs to the TypeScript output panel.
- Change activation behavior or supported file types: edit `package.json` and `src/client.ts`.

## Example patterns to reference

- Class component prop discovery: `template({ todo, onToggle, onRemove })`
- Function component prop discovery: `function TodoFilters({ filter, activeCount, completedCount, onFilter })`
- `props` destructuring discovery: `function PaymentForm(props) { const { passengerName, onPay } = props }`
- TS plugin behavior: see `filterDiagnostics()` in `ts-plugin/gea-template-plugin.js`

## Practical suggestions for the AI agent

- Keep imports extensionless in TypeScript files.
- Prefer improving component discovery over hardcoding completions.
- Keep the built-in component list in sync with `@geajs/mobile`.
- Avoid changing the `contributes.typescriptServerPlugins` entry without updating the plugin code too.

## Build/quality gates to run locally

- `npm run compile` — TypeScript build (required before F5).
- Open Extension Development Host (F5) and check:
  - TypeScript output panel (plugin logs)
  - Language Server output (client/server console logs)
  - That completions appear in JSX tags and props
  - That unknown component diagnostics appear for intentionally broken JSX tags

## If anything is unclear

Please tell me which sections you want expanded (examples, more file pointers, or a checklist for making parser/plugin changes). I'll iterate the instructions accordingly.

---
> Source: [dashersw/gea](https://github.com/dashersw/gea) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-24 -->
