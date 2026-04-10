# Project Guidelines

## Commit Messages

Use conventional commits format in English.

### Format

**Simple change** — single line:
```
<type>: <short description>
```

**Complex change** — header + body:
```
<type>(<scope>): <short description>

- detail 1
- detail 2
```

### Types
- `feat`: new feature
- `fix`: bug fix
- `refactor`: code restructuring
- `style`: formatting, whitespace (no logic change)
- `docs`: documentation
- `chore`: tooling, config, dependencies
- `perf`: performance improvement
- `test`: adding or updating tests

### Rules

1. Keep it proportional to the change:
   - Typo fix → `fix: typo in component name` (that's it)
   - Simple rename → `refactor: rename X to Y`
   - New feature with multiple parts → header + bulleted body

2. The header must be short enough to read at a glance in GitHub/git log.

3. When a commit includes many changes, focus the message on what matters. Omit trivial side-effects (formatting, typos, minor cleanup) from the description — unless the commit is specifically about those things.

4. No need to list every file touched. Describe the intent, not the diff.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/notvincent8)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/notvincent8)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
