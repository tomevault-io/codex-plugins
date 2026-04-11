# ChronoClean – Copilot Instructions

This repository is largely AI-assisted.
The following rules exist to prevent architectural drift, accidental complexity,
and silent code duplication.

Copilot must follow these rules strictly.

---

## 1. Architecture

- Respect the project architecture at all times.
- One business rule = one implementation.
- Do not duplicate logic across modules.
- Prefer simple, explicit code over generic abstractions.
- Do not introduce new abstractions unless reuse already exists.

Refer to `ARCHITECTURE.md` for the authoritative architecture contract.

---

## 2. Duplication policy

- Intra-file duplication must be refactored immediately.
- Inter-module duplication is forbidden.
- Any refactor must reduce or preserve the current duplication level.
- `jscpd` is the reference tool for duplication detection.

If duplication exists, refactor before adding new code.

---

## 3. Constants and utilities

- `utils/constants.py` contains values only.
  - No functions.
  - No conditionals.
  - No logic.
- Utilities must solve a real duplication (used in at least two places).
- Do not create speculative helpers or “utils dumping ground”.

---

## 4. Function complexity

- Functions should generally stay under ~40 lines.
- Excessive branching or local variables must be justified.
- Refactor locally before introducing new modules or classes.

---

## 5. CLI-specific rules

- CLI code is orchestration code.
- Higher complexity is acceptable in `chronoclean/cli/**`.
- Business logic must never live in CLI modules.
- Core logic must remain reusable outside the CLI.

---

## 6. Refactoring discipline

- Prefer small, targeted refactors.
- Do not mix refactoring with feature changes unless explicitly requested.
- Preserve existing behavior unless instructed otherwise.

---

## 7. Tooling awareness

The following tools define the structural baseline of the project:

- `jscpd` → duplication guardrail
- `pylint` → structural warnings
- `radon` → complexity visibility

Warnings are signals, not dogma.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Stilmant)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Stilmant)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
