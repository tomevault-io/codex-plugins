<!-- Copilot instructions for working on the renpy-lsp-vscode extension -->

# Guidance for AI coding agents

This repo implements a VS Code extension that starts a small Python-based language server (LSP) for Ren'Py (.rpy) files. Below are concise, actionable notes to help AI coding agents be productive immediately.

- **Big picture:** The extension (TypeScript) is the VS Code client and ships a bundled Python LSP server. The client lives in [src/extension.ts](src/extension.ts) and the server in [bundled/tools/lsp_server.py](bundled/tools/lsp_server.py). The parser used by the server is in [bundled/tools/ast_parser.py](bundled/tools/ast_parser.py).

- **Server module structure (`bundled/tools/`):**
  - `lsp_server.py` — main entry point; LSP feature handlers, diagnostics, document sync, cache management.
  - `ast_parser.py` — indentation-aware parser producing AST nodes (`Label`, `If`, `Say`, etc.).
  - `renpy_data.py` — pure-data constants with zero runtime dependencies: `RENPY_KEYWORDS`, `RENPY_TRANSITIONS`, `RENPY_TRANSFORMS`, `KEYWORD_DOCS`, `count_words()`.
  - `workspace_index.py` — `WorkspaceIndex` class; incrementally maintained index of labels/defines/screens/images across workspace `.rpy`/`.rpym` files. Receives its dependencies (cache, server, utility fns) via constructor injection to avoid circular imports.

- **How it runs locally:** The client expects a Python runtime in a virtualenv at `.venv/bin/python3.11` and launches the server script with stdio transport. See `startLanguageServer` in [src/extension.ts](src/extension.ts#L1-L200).

- **Build & test (JS/TS side):**
  - Compile: `npm run compile` (runs `webpack`) — produces `dist/extension.js` per `package.json`.
  - Watch: `npm run watch` for incremental builds.
  - Run tests: `npm run test` (runs the test runner in `out/test/runTest.js`).

- **Python environment & deps (server):**
  - `pyproject.toml` and `requirements.txt` require Python >=3.11 and packages `pygls` and `lsprotocol`.
  - To run server manually: install deps into `.venv` using Python 3.11, then run `./.venv/bin/python3.11 bundled/tools/lsp_server.py`.

- **Key patterns & conventions:**
  - Activation: extension auto-starts when `.rpy` files exist (`activationEvents` in [package.json](package.json)).
  - Server launch: the extension uses `TransportKind.stdio` to start the Python process; do not change to socket without updating both ends. See `serverOptions` in [src/extension.ts](src/extension.ts#L60-L110).
  - Formatting: the server implements full-document formatting and returns a single TextEdit replacing the entire document (see `[bundled/tools/lsp_server.py](bundled/tools/lsp_server.py#L1-L200)`). Agents making changes should preserve that contract.
  - AST: `bundled/tools/ast_parser.py` is an indentation-aware parser producing simple AST nodes (`Label`, `If`, `Say`, etc.). Use it as the canonical parser for language features.
  - Module extraction: `renpy_data.py` holds pure constants (keyword lists, hover docs, word-count helper). `workspace_index.py` holds the `WorkspaceIndex` class using dependency injection — it never imports from `lsp_server.py`; instead, `lsp_server.py` passes cache/utility references via the constructor.

- **Common tasks examples:**
  - Add a new LSP feature (e.g., hover): implement handler in `bundled/tools/lsp_server.py` using `@LSP_SERVER.feature(...)`, then register relevant client capabilities in `clientOptions` in [src/extension.ts](src/extension.ts#L110-L160).
  - Change indentation rules: update [language-configuration.json](language-configuration.json) — `increaseIndentPattern`/`decreaseIndentPattern` are regex-based and important for editor behavior.

- **Debugging tips:**
  - To debug the Python server, run it manually in the `.venv` and attach logs to stdout; the client expects stdio comms.
  - When the extension fails to start the server, check that `bundled/tools/lsp_server.py` exists and that the configured Python binary in [src/extension.ts](src/extension.ts#L60-L80) is executable.

- **Files to review first:** [src/extension.ts](src/extension.ts), [bundled/tools/lsp_server.py](bundled/tools/lsp_server.py), [bundled/tools/ast_parser.py](bundled/tools/ast_parser.py), [bundled/tools/renpy_data.py](bundled/tools/renpy_data.py), [bundled/tools/workspace_index.py](bundled/tools/workspace_index.py), [language-configuration.json](language-configuration.json), and [package.json](package.json).

If any runtime assumptions (Python path, venv layout, or activation behavior) are wrong or you need additional CI/build commands added, tell me what should be discovered or changed and I will update this file.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Zhuxb-Clouds) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
