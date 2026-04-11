# Copilot Instructions

## General

- You have extensive expertise in Svelte 5, Svelte Kit, TypeScript, Vite, and CSS.
- You possess a deep knowledge of best practices and performance optimization techniques across these technologies.

## Code Style and Structure

- Write clean, maintainable, and technically accurate TypeScript code.
- Prioritize functional and declarative programming patterns; avoid using classes.
- Emphasize iteration and modularization to follow DRY principles and minimize code duplication.
- Write JSDoc comments for all exported functions, variables, and types for better readability and understanding.
- In JSDoc, `@return` annotation can be omitted since it's self-explanatory in most cases.

## HTML

- Specify `type` attribute for `<button>` elements.

## Svelte 5 Specifics

- Use runes API introduced in Svelte 5.

## TypeScript Usage

- Prefer function declarations over function expressions. However, when passing a function to a parameter, prefer using a function expression.
- Define function return types for better readability and type safety.
- Use literal union types over enums.
- Use `unknown` over `any`.
- Prefer `undefined` over `null` unless `null` has to be used.
- Prefer `for...of` statement over `forEach`. However, allow use of `forEach` when chaining.
- Avoid non-null assertion operator (`!`) unless absolutely sure the value cannot be `null` or `undefined`.
- Put `as const` on constant objects.
- Prefer the guard clause (early return) when the rest of the function can be skipped after the return.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Tsuyoshi84)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Tsuyoshi84)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
