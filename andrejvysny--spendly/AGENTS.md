
# Commands

- **Backend:** `php artisan test`, `./vendor/bin/phpstan analyse`, `./vendor/bin/pint`
- **Frontend:** `npm run dev`, `npm run test`, `npm run types`, `npm run lint`, `npm run format:check`
- **Full stack:** `./scripts/test.sh` (Docker)
- Prefer targeted runs: `php artisan test --filter=ClassName`, `npm test -- path/to/file`

# Workflow

- After PHP changes: run phpstan and pint (or tests).
- After TS/React changes: run types and lint (or tests).
- Modifiable vs protected dirs: see AGENTS.md.

---
> Source: [andrejvysny/spendly](https://github.com/andrejvysny/spendly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-19 -->
