# TestDriver — Copilot Instructions

## Build Commands
| Target | Command |
|--------|---------|
| Build | `ant build` |
| Test | `ant test` |
| Clean | `ant clean` |
| Debug | `ant build -Ddebug=true` — outputs to `debug/` instead of `build/` |

## Build Order
framework → shared-utils → driver
Reason: each step produces JARs consumed by the next. Skip = compile errors.

## Boundaries
- NEVER modify files under `shared/` without cross-project regression testing
- NEVER commit `.env`, `credentials.properties`, or DSN configuration files
- NEVER change public API method signatures without explicit approval
- ASK FIRST before modifying `framework/` — changes affect ALL driver projects

## Conventions (Quick Reference)
<!-- Summary of shared/ v2026.1 -->
- Error creation: always use `BaseExceptions.getException()`, never `new SQLException()` directly
- Logging: use `DDLogBuilder`, never `System.out.println` or `java.util.logging` directly
- All public method names MUST end with suffix "ZQ" (e.g., `connectZQ`, `executeZQ`, `closeZQ`)
- Resource cleanup: explicit try/finally — no try-with-resources in production code
- Java version: Java 8 only — no var, records, sealed classes, text blocks

## Security (Quick Reference)
<!-- Summary of shared/ v2026.1 -->
- NEVER log connection strings, passwords, or auth tokens — emit `"********"` instead
- NEVER store credentials in plaintext — zero out after use
- NEVER commit secrets, keys, or tokens to the repository
- ASK FIRST before changing TLS/SSL, keystore, or encryption settings
- Reference: `.github/instructions/shared/universal/security.instructions.md`

## Code Review Behavior
- Only comment when HIGH confidence (>80%) that an issue exists
- Do NOT comment on style or formatting — linters handle that
- Do NOT suggest refactoring unless there is a clear bug or security issue

## Do Not Review
- Do NOT suggest Java 9+ features — Java 8 max is deliberate
- Do NOT suggest replacing `BaseExceptions.getException()` with standard `new SQLException()`
- Do NOT suggest replacing `DDLogBuilder` with SLF4J or other logging frameworks

## Review Focus Areas
- Resource leaks (connections, statements, result sets not closed)
- Exception handling (raw exceptions vs. BaseExceptions.getException())
- Thread safety in connection pool and statement code
- Credential exposure in logging or configuration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/saivenkat999)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/saivenkat999)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
