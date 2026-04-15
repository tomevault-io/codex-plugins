# Go Payment — Project Guidelines

## Overview

Offline-first NFC card-based payment system for Nepal. See `PLANNING.md` for full architecture, `IMPLEMENTATION.md` for sprint plan.

## Architecture

- **Card** = Offline cash chip (DESFire EV2/EV3, AES-128)
- **POS** = Trusted but limited executor (Kotlin Android)
- **Server** = Auditor + Float ledger (Go single binary)
- **HSM** = Root of trust (SoftHSM2 dev, YubiHSM2 prod)

The server is NOT a payment processor. All payment authority lives on card + POS. Server reconciles after the fact.

## Tech Stack

| Component | Technology |
|-----------|-----------|
| Backend | Go (single binary), Gin/Echo, PostgreSQL 16 |
| POS App | Native Kotlin + Jetpack Compose, Android 8+ |
| Admin Dashboard | Next.js 15 + shadcn/ui + Tailwind |
| Companion App | React Native (Expo) — Phase 3 |
| Deployment | Docker Compose, Nginx, Hetzner VPS |
| CI/CD | GitHub Actions or Gitea + Drone |

## Repository Structure

```
cmd/server/       — Main API server (single binary)
cmd/cardtool/     — Card provisioning CLI (HSM-connected stations only)
internal/         — All business logic (auth, card, float, transaction, merchant, terminal, agent, concession, settlement, fraud, compliance, sync, jobs, common)
pkg/desfire/      — DESFire protocol library
api/openapi/      — OpenAPI 3.0 specs
migrations/       — PostgreSQL migrations (float ledger first)
deployments/      — Docker Compose + Nginx
web/admin/        — Next.js admin dashboard
pos/              — Android POS app (Kotlin)
app/              — React Native companion app (Phase 3)
```

## Core Principles

1. **Offline first** — every transaction works without internet
2. **Card first, app optional** — NFC card is primary interface
3. **Zero paid third-party dependencies** — self-hosted or OSS only
4. **Float integrity** — double-entry accounting, 1:1 backing provable at all times
5. **Tamper resistant** — AES-128 mutual auth, transaction MACs, hardware-enforced limits

## Code Conventions

### Go (Backend)
- Single binary architecture — no microservices in Phase 1
- All money amounts in **paisa** (bigint) — never float
- Double-entry ledger: every money movement = debit + credit entries
- Idempotency keys on all state-changing operations
- PostgreSQL for everything including job queue (no Redis/NATS until >5K POS)
- Structured JSON logging, no PII in plain text
- `internal/` for all business logic, `pkg/` only for reusable libraries

### Kotlin (POS)
- Native Android, no Flutter/React Native — hardware NFC + keystore access required
- Hardware-backed keystore (StrongBox) for device keys
- Offline-first: all card operations work without network
- Transaction counter is first-class — read before, verify after every txn
- Room DB for local transaction ledger

### Next.js (Admin Dashboard)
- App Router, shadcn/ui components, Tailwind
- Server components by default, client components only when needed

### Database
- Migrations ordered by dependency: float tables → entities → transactions → compliance → jobs
- All migrations must be reversible (up + down)
- `float_accounts.entity_type`: agent | merchant | system_reserve | escrow | clearing
- Critical invariant: `SUM(card_balances) = system_reserve = escrow_bank_actual`

## Security Rules

- Card keys diversified per-card using AN10922 (master key never on card)
- Key hierarchy: ROOT_MASTER_KEY → CARD_MASTER_KEY / TERMINAL_MASTER_KEY / MAC_MASTER_KEY
- ROOT_MASTER_KEY lives in HSM only — never in application memory
- Device attestation validated server-side on every sync (not just client-side)
- No sync → no settlement (server-side enforcement)
- Provisioning tool (`cmd/cardtool/`) is CLI-only, no network API, no remote invocation

## Build & Test

```bash
# Backend
docker compose up -d          # PostgreSQL + SoftHSM2
go build ./cmd/server/
go test ./internal/... -race

# Admin dashboard
cd web/admin && pnpm install && pnpm dev

# POS
cd pos && ./gradlew assembleDebug
```

## Key References

- `PLANNING.md` — Full architecture, threat model, regulatory compliance, DB schema
- `IMPLEMENTATION.md` — Sprint-level plan, acceptance criteria, milestones

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nischaltimalsina) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
