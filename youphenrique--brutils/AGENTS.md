# GitHub Copilot Instructions

## Project Overview

`brutils` is a TypeScript monorepo of utilities for Brazilian business rules and data validation. The published package is `@brutils/core` (in `packages/core/`), which currently covers CPF, CNPJ, and Brazilian states (UFs).

## Toolchain: Vite+ (`vp`)

All development tasks go through the `vp` CLI — **do not use pnpm, npm, vitest, oxlint, or tsdown directly**.

| Task                       | Command                                               |
| -------------------------- | ----------------------------------------------------- |
| Install dependencies       | `vp install`                                          |
| Format                     | `vp fmt`                                              |
| Lint (type-aware)          | `vp lint`                                             |
| Type-check + format + lint | `vp check`                                            |
| Run all tests              | `vp test` (inside `packages/core/`)                   |
| Run a single test file     | `vp test cpf` (name pattern passed to Vitest)         |
| Build library              | `vp pack` (inside `packages/core/`)                   |
| Full validation pipeline   | `pnpm run ready` (from root — fmt, lint, test, build) |

> `vp test` and `vp build` run the Vite+ built-ins, **not** `package.json` scripts of the same name. To run a custom script that shares a name with a built-in, use `vp run <script>`.

## Imports

Always import from `vite-plus`, never from `vite` or `vitest` directly:

```ts
import { defineConfig } from "vite-plus";
import { expect, test, vi } from "vite-plus/test";
import { defineConfig } from "vite-plus/pack"; // for tsdown.config.ts
```

## Architecture

`packages/core/src/` is organized into feature modules. Each module follows a strict internal structure:

```
src/<module>/
  constants.ts   # Exported constants (e.g., LENGTH, STATES_REGION_MAP)
  types.ts       # TypeScript types and interfaces
  utils.ts       # Internal helpers + the module's Error class
  <module>.ts    # Public functions (validate, format, generate, normalize, mask)
  index.ts       # Re-exports everything public
```

The root `src/index.ts` re-exports each module as a namespace:

```ts
export * as cpf from "./cpf/index";
export * as cnpj from "./cnpj/index";
export * as ufs from "./ufs/index";
```

Shared utilities live in `src/_shared/` (e.g., `assert-options.ts`).

## Key Conventions

### Validate functions return result objects, never throw

```ts
// ✅ Correct pattern
validate("123"); // { success: false, error: CpfError("INVALID_FORMAT") }

// Throws only for wrong input type
validate(123); // throws TypeError
```

### Error classes have a `code` property

Each module defines its own Error class extending `Error` with a `readonly code` typed as a union of string literals:

```ts
export class CpfError extends Error {
  readonly code: CpfErrorCode; // "INVALID_FORMAT" | "REPEATED_DIGITS" | ...
  constructor(code: CpfErrorCode, message?: string) { ... }
}
```

### Internal `preNormalize` + `assertValid` pattern

Public `validate()` functions delegate to:

1. `preNormalize(value)` — enforces a strict format (raw digits OR standard mask only); throws `ModuleError("INVALID_FORMAT")` for anything else.
2. `assertValid(normalized)` — checks business rules (checksum, repeated digits, etc.); always throws on failure.

### Options validation

Any function that accepts an options object must call `assertOptions(options)` from `src/_shared/assert-options.ts` before destructuring:

```ts
import { assertOptions } from "../_shared/assert-options";

export function generate(options: CpfGenerateOptions = {}): string {
  assertOptions(options);
  const { state, formatted = false } = options;
  // ...
}
```

### TypeScript strictness

`packages/core/tsconfig.json` enables `strict`, `noUnusedLocals`, `verbatimModuleSyntax`, and `isolatedModules`. Use `import type` for type-only imports.

## Adding a New Utility Module

1. Create `src/<module>/` with all five files: `constants.ts`, `types.ts`, `utils.ts`, `<module>.ts`, `index.ts`.
2. Add the Error class in `utils.ts` following the existing pattern.
3. Export the module namespace from `src/index.ts`.
4. Add tests in `tests/<module>.test.ts`.
5. Run `pnpm run ready` from the root to validate everything.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/youphenrique)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/youphenrique)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
