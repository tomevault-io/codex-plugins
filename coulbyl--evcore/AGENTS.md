# GitHub Copilot Instructions — EVCore

EVCore is an autonomous sports betting engine based on Expected Value (EV) mathematics. Read `EVCORE.md` for the full specification and `CLAUDE.md` for coding conventions.

## Stack

- **Runtime**: Node.js >= 18, TypeScript (strict)
- **Backend**: NestJS + Prisma + PostgreSQL
- **Queues**: BullMQ + Redis
- **Validation**: Zod (external data) + class-validator (NestJS DTOs)
- **Math**: `decimal.js` for odds/EV, `jStat` for Poisson distributions
- **Notifications**: Nodemailer (SMTP) + in-app (PostgreSQL)
- **Monorepo**: pnpm + Turborepo

## Architecture Rules

- Controllers handle routing and DTO validation only — no business logic
- Services contain all business logic — they call repositories, never Prisma directly
- Repositories contain all Prisma queries — one per model
- Zod validates all external data (ETL, API responses, OpenClaw output) before any DB write
- `class-validator` validates all inbound HTTP DTOs

## Naming Conventions

- Files: `kebab-case.ts`
- Classes/Types: `PascalCase`
- Functions/variables: `camelCase`
- Constants: `UPPER_SNAKE_CASE`
- DB tables: `snake_case` (Prisma default)

## Critical Rules — Never Break These

- Never use native `number` for odds, EV, or bankroll — always use `decimal.js`
- Never hardcode `0.08` (EV threshold), `0.30` (LLM cap), `50` (min bets), `5%` (max weight change) — import from `config/`
- Never call Prisma directly outside a `*.repository.ts` file
- Never let OpenClaw output skip Zod validation
- Never auto-apply an `AdjustmentProposal` — backend decision only
- Never reactivate a suspended market automatically
- Never use LLM to fetch, infer, or fill in fixture or odds data
- Never implement Kelly criterion (Phase 2 only, behind config flag)
- Probability values must always be in `[0, 1]` — assert at ingestion boundary
- Never read or expose `.env` / `.env.*` files (except `.env.example`)
- Never log, commit, or print secrets (API keys, passwords, tokens, DB URLs)

## EV Formula

```ts
// Always use decimal.js — never native arithmetic
import Decimal from "decimal.js";

function calculateEV(probability: Decimal, odds: Decimal): Decimal {
  return probability.mul(odds).minus(1);
}
```

## ETL Pattern

Every ingestion job must:

1. Fetch raw data
2. Validate with Zod schema (reject entirely if invalid — never partial writes)
3. Log failed payloads with full context
4. Write to DB only after successful validation
5. Trigger notification alert on total failure

## ModelRun Output

Every bet decision must log: `fixture_id`, `features` (all 4), `deterministic_score`, `llm_delta`, `final_score`, `decision`, `validated_by_backend`.

## Phase Awareness

Do not implement features from future phases without explicit instruction:

- MVP: historical import, backtest, calibration
- Phase 2: odds, EV live, OpenClaw, Grafana, TimescaleDB
- Phase 3: Python worker, ML model, Monte Carlo
- Phase 4: SaaS, multi-tenant

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/coulbyl) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
