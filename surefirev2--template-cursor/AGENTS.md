# Semantic Commit Message Rule

All commit messages must follow the [semantic commit message](mdc:https:/www.conventionalcommits.org/en/v1.0.0) standard. This ensures clarity, consistency, and enables better automation in the project.

## Format
- `<type>(optional scope): <description>`
- Valid types: `feat`, `fix`, `chore`, `docs`, `refactor`, `test`, `style`, `perf`, `ci`, etc.
- The description should be concise and use the imperative mood.

## Examples
- `feat: add setup script for environment and hooks`
- `fix(setup): prevent double .sh extension when migrating templates`
- `docs(README): update setup instructions`

## Consequences
- Commits that do not follow this rule may be rejected or require rewording before merging.
- Automated tools and workflows may depend on this format for changelogs and releases.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/surefirev2)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/surefirev2)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
