---
description: Package manager rules for Zombie.Digital project - always use bun instead of npm/yarn/pnpm
alwaysApply: true
---

# Package Manager Rules

Always use bun for package management in this project.

## Commands to use:
- `bun add package-name` for package installation
- `bunx package-name` for package execution  
- `bun run script-name` for running scripts

## Commands to avoid:
- ❌ `npm install`
- ❌ `npx package-name`
- ❌ `yarn add`
- ❌ `pnpm add`

## Examples:
```bash
# ✅ Correct
bun add @types/node
bunx create-next-app
bun run dev

# ❌ Incorrect
npm install @types/node
npx create-next-app
npm run dev
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/joenilan)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/joenilan)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
