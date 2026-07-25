# Copilot Instructions — lite-server

## Project Type

This is a **lightweight npm package** — a development server wrapping BrowserSync for SPAs. Pure JavaScript, no build step, no TypeScript.

## Writing Conventions

### JavaScript

- Use `'use strict'` at the top of every file
- Use single quotes for strings
- Use 2-space indentation
- Use `const` by default; `let` only when reassignment is needed
- Use camelCase for variables and functions
- Use 1TBS brace style (`if (x) {`)
- Max line length: 100 characters
- Use `===` for equality checks (enforced by ESLint `eqeqeq` rule)
- Prefer arrow callbacks (`prefer-arrow-callback` rule)
- Prefer template literals over string concatenation (`prefer-template` rule)
- Semicolons required

### Testing

- Test files live in `test/` with `.spec.js` suffix
- Use Mocha `describe`/`it` blocks
- Use `mockery` for module mocking with `useCleanCache: true`
- Use `sinon` for stubs and spies
- Use Node.js built-in `assert` for assertions (not chai or other libraries)
- Always clean up mockery in `afterEach` with `deregisterAll()` and `disable()`
- Register allowed modules with `mockery.registerAllowable()`

### Project

- No build step — ship raw JavaScript
- Entry point: `index.js` (public API), `bin/lite-server` (CLI)
- Core logic: `lib/lite-server.js`
- Default config: `lib/config-defaults.js`

## Commands

```bash
npm install    # Install dependencies
npm test       # Run ESLint + Mocha/Istanbul coverage
```

## Maintenance Matrix

| When this changes... | Also update... |
|---|---|
| `lib/lite-server.js` (core server logic) | `test/lite-server.spec.js`, `README.md` (if behavior changed), `CHANGELOG.md` |
| `lib/config-defaults.js` (default config) | `test/config-defaults.spec.js`, `README.md` (if defaults changed), `CHANGELOG.md` |
| `package.json` (dependencies, scripts) | `CHANGELOG.md`, verify `npm test` still passes |
| `bin/lite-server` (CLI entry) | `README.md` (if CLI usage changed), `CHANGELOG.md` |
| `.eslintrc` (lint rules) | Verify `npm test` still passes, `CHANGELOG.md` |
| `index.js` (public API exports) | `README.md` (if API changed), `CHANGELOG.md` |
| New middleware added | `lib/config-defaults.js`, `test/config-defaults.spec.js`, `README.md` |

---
> Source: [johnpapa/lite-server](https://github.com/johnpapa/lite-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-24 -->
