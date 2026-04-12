# Harness Engineering — Windsurf Rules

Read `AGENTS.md` at the project root first. This file extends it for Windsurf / Cascade.

## Session protocol
1. `bash init.sh`
2. Read last 30 lines of `claude-progress.txt` and parse `features.json`
3. Check `agents.json` — register session, claim highest-priority failing feature
4. Implement exactly one feature per session

## Skills
Reference with `@skill-name` in Cascade chat:
`@session-start` `@session-end` `@implement-feature` `@new-feature` `@run-tests`
`@create-issue` `@delegate-subagent` `@entropy-check` `@write-adr`

## Rules
- No WIP on `main`. One feature per session. No scope expansion.
- Layer order: `Types → Config → Repo → Service → Runtime → UI`
- Blocked: `rm`, `del`, `rmdir`, `kill`, `sudo`, `&&`, `||`
- Update `claude-progress.txt` and `agents.json` before ending session

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ArtemisAI)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/ArtemisAI)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
