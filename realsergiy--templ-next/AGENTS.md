# Coding Agent Instructions (RFC‑2119)

> **Conventions**: Each rule begins with a severity tag and a bold modal.  
> Severities: `[P0]` non‑negotiable · `[P1]` strong preference · `[P2]` nice‑to‑have.  
> Modals: **MUST**, **MUST NOT**, **SHOULD**, **SHOULD NOT**, **MAY**.  
> Bold is used **only** on the modal. Code/config appear in backticks.

---

## All Languages

- [P0] **MUST NOT** fabricate facts or behavior; if uncertain, ask a brief clarifying question
  before large changes.
- [P0] **MUST NOT** disable, suppress, or modify linter/formatter rules to bypass violations;
  **SHOULD** fix the underlying code to comply with configured rules.
- [P1] **MUST** align with repository tooling: produce output that passes configured
  formatters/linters/tests and CI where applicable.

---

## Comments

- [P0] **SHOULD NOT** comment obvious operations, getters/setters, or standard library/standard
  functions.
- [P0] **MUST NOT** communicate with users through code comments; use non‑code channels available in
  the current environment (e.g., chat UI, console output, logs, pull‑request description).
- [P0] **SHOULD** preserve user comments marked with `// ToDo:`.
- [P1] **SHOULD** write comments only for complex algorithms or non‑obvious business logic.
- [P0] **SHOULD** focus comments on _why_ decisions were made, not _what_ the code does.

---

## Markdown (markdownlint)

- [P1] **MUST** follow markdownlint rules from `.markdownlint.json`.
- [P1] **MUST** generate Markdown that passes markdownlint with zero warnings.

---

## TypeScript

### Compiler & Strictness

- [P0] **MUST** enable `strict`, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`,
  `erasableSyntaxOnly`, `forceConsistentCasingInFileNames`, `verbatimModuleSyntax`.
- [P0] **MUST NOT** use `any`; use `unknown` or proper types.
- [P0] **MUST NOT** use `@ts-ignore`, `@ts-nocheck`, or `@ts-expect-error`.
- [P1] **MUST NOT** use non-null assertion operator (`!`); **MUST** handle potential
  `undefined`/`null` values explicitly with proper checks or type guards.

### Type Inference

- [P0] **MUST NOT** add type annotations anywhere TypeScript can infer them correctly.
- [P0] **MUST NOT** add function return type annotations and variable type annotations if they can
  be inferred.
- [P0] **MUST** remove all redundant type annotations when refactoring existing code.
- [P0] **SHOULD** rely on TypeScript's inference by default.

### Type Safety

- [P0] **MUST** handle all `undefined`/`null` cases explicitly.
- [P0] **SHOULD** use type guards for narrowing; **SHOULD NOT** rely on type assertions (`as`).
- [P2] **MUST** use strict equality (`===`, `!==`) exclusively for predictability.

### Type Definitions

- [P0] **MUST** use `type` aliases for new types; **MUST NOT** use `interface`.
- [P0] **SHOULD** prefer `readonly` fields and `ReadonlyArray` for immutability.
- [P1] **SHOULD** use discriminated unions for polymorphic shapes.
- [P2] **SHOULD** avoid declaration merging and module augmentation; keep types explicit and
  traceable.
- [P2] **SHOULD** extract complex composite types into named aliases.
- [P2] **SHOULD** define types for all data structures and domain models.

### Code Style

- [P0] **SHOULD** prefer arrow functions over function declarations.
- [P0] **SHOULD** use functional programming patterns.
- [P0] **SHOULD** embrace latest modern TypeScript patterns and features.
- [P0] **SHOULD** use top‑level `await` for async entry points; **SHOULD NOT** use
  `.then()/.catch()` chains or module‑level IIFEs.
- [P1] **MUST NOT** mark a function `async` unless it contains `await` or explicitly returns a
  `Promise`.
- [P2] **MAY** use `const` assertions for literal types when helpful.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/realSergiy)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/realSergiy)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
