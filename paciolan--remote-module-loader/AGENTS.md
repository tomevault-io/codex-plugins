# CLAUDE.md

## Project Overview

`@paciolan/remote-module-loader` is a TypeScript library that dynamically loads CommonJS modules from remote URLs. It works in both browser (XMLHttpRequest) and Node.js (http/https) environments, auto-detecting which fetcher to use. Modules are evaluated via `new Function()` with dependency injection through a `requires` function.

---
> Source: [Paciolan/remote-module-loader](https://github.com/Paciolan/remote-module-loader) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-24 -->
