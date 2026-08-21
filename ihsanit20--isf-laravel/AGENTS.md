# ISF Laravel Instructions

This repository powers the backend, admin workflows, reporting, and APIs for Al-Ihsan Savings Fund (ISF).

Product and domain priorities:

- ISF starts as a voluntary monthly office group savings fund focused on disciplined savings, transparency, and member trust
- Treat the fund-cycle model as the active product direction: deposit, unallocated balance, member allocation, cycle lock, investment, maturity, and settlement
- Keep fund-cycle money, member-oriented charges, and extra-fund expenses clearly separated in workflow and reporting
- Support member-wise participation and settlement traceability at the fund-cycle level
- Do not introduce dividend logic, speculative abstractions, or complex loan logic unless explicitly requested
- Qard-e-hasana is a possible later feature and must stay behind a separate policy and workflow

Core rules:

- Treat `users` as login accounts
- Treat `members` as fund memberships
- One user may manage multiple members, including family memberships if the product later enables it
- All financial records must be member-based, not user-based
- Members may hold one or more units; keep unit counts explicit and simple
- Use 1000 BDT as the default monthly unit amount unless the user requests otherwise
- Participation is voluntary, and membership should not be auto-cancelled because of job change, relocation, or leaving the company
- If exit flows are implemented, preserve clear settlement states and support a settlement window such as 30 to 90 days

Money handling and auditability:

- Prefer bank-channel-based deposits and settlements; avoid cash-first designs
- Store deposit submissions with payer-entered proof and admin verification status
- Preserve approver identity, timestamps, references, and notes for every financial action
- Do not hard-delete financial records
- Prefer append-only ledger history and adjustments over editing approved historical transactions

Fund-cycle database baseline:

- Keep the current minimal fund-cycle schema unless the user explicitly asks to revisit it
- Use existing global `deposit_submissions` as the deposit source; do not introduce `fund_cycle_deposits` unless explicitly requested
- Do not introduce `fund_cycle_members` while cycle participation can be derived from `fund_cycle_allocations`
- Minimal fund-cycle tables are `fund_cycles`, `fund_cycle_allocations`, `fund_cycle_investments`, `fund_cycle_returns`, and `fund_cycle_settlements`

Planning guardrails:

- Do not give a new database or product plan from broad full-project analysis by default
- Before proposing changes to an agreed plan, first anchor on the existing local implementation and previously agreed repository direction
- Only revisit the saved fund-cycle plan when the user explicitly asks to change, review, or replace it

Laravel implementation guidance:

- Follow Laravel conventions for models, controllers, form requests, policies, migrations, factories, seeders, and tests
- Use Form Request validation for non-trivial input
- Keep controllers thin; move business logic out when it becomes reusable or materially complex
- Prefer explicit status fields and direct domain names over generic abstractions
- When schema changes affect money flow, also review factories, seeders, policies, validation, notifications, and tests

Admin and member workflow expectations:

- Admins should be able to review submitted deposit proof and approve or reject it with a traceable decision
- Members should be able to see balances, contributions, deposit statuses, and ledger history clearly
- Reports should prioritize transparency, current balance visibility, and reconciliation support

Decision principle:

- Optimize for traceable financial data, simple maintenance, clear member workflows, low operational risk, and room to extend later without overbuilding now

---
> Source: [ihsanit20/isf-laravel](https://github.com/ihsanit20/isf-laravel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-21 -->
