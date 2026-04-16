# Copilot Instructions — Weapons of Order (WOO)

> **These instructions are MANDATORY for all AI agents working on this project.**

---

## Project Identity

WOO is a **server-authoritative multiplayer strategy game** (browser-based, Travian-inspired). Players build villages, train troops across 8 kingdoms, and collaborate to defeat the NPC faction Moraphys.

Start every task by reading `docs/00-getting-started/project-overview.md`.

---

## Three Architectural Pillars

1. **Server-authoritative**: ALL game logic runs server-side. Client sends intents, server validates and executes. Never trust the client for gameplay-affecting calculations.
2. **Lazy resource calculation**: Resources are calculated on read (`stored + rate × elapsed`), written only on events. Never tick/update resources periodically.
3. **Config codegen pipeline**: Game config lives in Go (`server/internal/config/`), auto-generated to JSON (`client/src/config/generated/`), imported by TS wrappers. **Never edit generated JSON directly** — edit Go source, run `npm run gen-config`.

---

## Documentation-First Workflow

**Before work**: Read docs relevant to your task (see table below).
**After work**: Update any docs your changes affect. Add a changelog entry (date + description) at the bottom.
**Before creating a new doc**: Search all existing docs first — prefer adding sections to existing files.

| Task area | Read |
|-----------|------|
| Frontend | `docs/04-frontend/` |
| Backend | `docs/05-backend/` |
| Database / migrations | `docs/06-database/` |
| Testing | `docs/07-testing/` |
| Security | `docs/08-security/` |
| Architecture | `docs/03-architecture/` |
| Game mechanics / features | `docs/01-game-design/` + `docs/02-lore/` |

When implementing kingdom-specific content, always cross-reference `docs/02-lore/` and `docs/01-game-design/kingdoms.md`.

---

## Core Rules

- **No code without tests.** Every feature needs unit tests. Multiplayer features need integration tests.
- **No hardcoded game values.** Use config, constants, or environment variables.
- **No console.log / fmt.Println.** Use `slog` (Go) or proper logging (TS).
- **File-specific conventions** are in `.github/instructions/` — they auto-load when you edit matching files.

---

## Lore Essentials

- **8 kingdoms**: Veridor (naval), Sylvara (forest), Arkazia (mountain), Draxys (desert), Nordalh (frost), Zandres (underground), Lumus (light), Drakanith (dragon — NPC-only).
- **Moraphys**: NPC enemy faction; triggers endgame by collecting all Weapons of Chaos.
- **Weapons of Chaos**: Configurable count per world (not hardcoded). Powerful but cause debuffs.
- **Weapons of Order**: Alliance-crafted counter to defeat Moraphys.
- **4 resources**: Food, Water, Lumber, Stone. 3 building slots each per village. Display names admin-configurable per kingdom.

---

## File Naming

| Type | Convention | Example |
|------|-----------|---------|
| React components | PascalCase | `VillagePanel.tsx` |
| Component styles | PascalCase + module | `VillagePanel.module.css` |
| Go files | snake_case | `village_handler.go` |
| Go test files | snake_case + _test | `village_handler_test.go` |
| SQL migrations | numbered + snake_case | `001_create_players.sql` |
| Documentation | kebab-case | `core-mechanics.md` |
| TypeScript types | PascalCase | `VillageTypes.ts` |
| Hooks | camelCase + `use` prefix | `useVillageData.ts` |
| Stores | camelCase + `Store` suffix | `villageStore.ts` |

---

## Git Workflow

- **Branches**: `master` (production), `develop` (integration), `feature/*`, `fix/*`, `docs/*`
- **Never push directly** to `master` or `develop`. Use feature branches.
- **Commits**: Conventional commits — `feat:`, `fix:`, `docs:`, `refactor:`, `test:`, `chore:`

---

## Dev Server Workflow

After completing any development task:

1. **Kill the running server**: `Get-Process -Name "server" -ErrorAction SilentlyContinue | Stop-Process -Force`
2. **If migrations changed**: Delete the database: `Remove-Item "d:\Workspace\WOO\server\data\woo.db*" -Force -ErrorAction SilentlyContinue`
3. **Rebuild and start**: `cd d:\Workspace\WOO\server; go build -o server.exe ./cmd/server; .\server.exe` (background)
4. **Start client**: `cd d:\Workspace\WOO\client; npm run dev` (background)

---

## Changelog

| Date | Change |
|------|--------|
| 2026-03-03 | Initial creation of copilot instructions |
| 2026-03-03 | Removed .mobile.css convention; added configurable Weapons of Chaos count |
| 2026-03-03 | Resources refactored to Food/Water/Lumber/Stone with 12 building slots |
| 2026-03-07 | Added Dev Server Workflow section |
| 2026-03-09 | Added Config Codegen Pipeline; updated kingdom list to 8 |
| 2026-03-12 | Major refactor: slimmed to essentials + links. Moved detailed conventions to `.github/instructions/` (go-backend, go-tests, frontend-react, css-modules, sql-migrations, config-codegen). Added prompts, agents, skills, and hooks. |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/luisfpires18) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
