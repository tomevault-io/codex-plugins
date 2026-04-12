# Claude Configuration Repository

This repo is symlinked to `~/.claude` and contains user-level Claude Code configuration. Changes here affect all projects on this machine.

## Key files

- `settings.json` - Global user settings (priority 5, committed to git)
- `settings.local.json` - Local machine-specific overrides (priority 3, gitignored)
- `agents/` - Custom agent definitions
- `commands/` - Custom slash commands

## Git rules

- **Never `git push` to remote without first running `/clean-settings`** to ensure `settings.json` does not contain sensitive or machine-specific data that belongs in `settings.local.json`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/davenicoll)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/davenicoll)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
