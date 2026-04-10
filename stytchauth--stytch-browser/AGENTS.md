
# Code Standards

## TypeScript

- Avoid `any`, type casting, and `@ts-expect-error` annotations
- All headless client methods must have JSDoc comments
- Use `@rbac` tag for RBAC info: `@rbac action="create", resource="stytch.member"`

## Documentation

- **Required**: All headless client methods and exports must have JSDoc comments
- **Source of Truth**: JSDoc comments are the primary source for API documentation
- **Method Descriptions**: Based on JSDoc blocks corresponding to method signatures

## Comments

Only add comments when code is unintuitive, complex, or handles unusual business requirements. Avoid comments that repeat what code does or echo user prompts.

## Internationalization

Lingui is used for i18n. Run `yarn strings` to extract and compile message catalogs after modifying translatable strings.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/stytchauth)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/stytchauth)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
