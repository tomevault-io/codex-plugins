
## TypeScript Rules

- Assume high TS/JS proficiency.
- Never use `any`.
  - If the value shape is known or derivable, create a **named** `interface` or `type` instead of `unknown`.
  - Use `unknown` only when truly generic; immediately narrow with guards or schema inference.
  - Mocks use `Partial<Type>`. Maps use `Record<string, ValueType>` with defined `ValueType`.
- Use interfaces for data structures; type aliases for unions/utility types.
- Prefer explicit return types for exported APIs.
- Prefer discriminated unions over boolean flags.
- Avoid non-null assertions; use guards/early returns.
- Use `as const` and `satisfies` to prevent widening.
- Optional chaining `?.` and `??`.
- Types/interfaces at end of component files; reusable non-prop types in `types.ts`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jrock2004)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/jrock2004)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
