# Copilot Instructions

## Project Guidelines
- Use dependency injection for anything that affects the external environment.
- When investigating and fixing failing tests, provide concrete code changes and validation rather than analysis only.
- Implement regex support using the .NET Regex library, allowing for deviations from canonical UNIX ed regex semantics.
- If a solution file is missing and project integration is needed in this workspace, create the solution file instead of skipping solution integration.

## Logging Guidelines
- Maintain a strictly sequential append-only history for LOG.md. Do not insert new entries into the middle of the log.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/enmasse)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/enmasse)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
