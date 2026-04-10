# AI Contributor Brief: gtc-form

## Scope & Architecture
- Static HTML frontends live under `chat/`, `auth/`, `verify/`, and `home/`; Nginx (`/etc/nginx/sites-enabled/app.gtstor.com`) serves them verbatim, so keep CSS/JS inline and dependency-free.
- Every flow revolves around `gtc_user_id`; its lifecycle is documented in `docs/chat_sql_persistence.md` and enforced by the SQL migration `db/migrations/20251125_chat_schema_actualization.sql` plus `create_all_tables.sql`.
- `chat/index.html` (~2k lines) owns auth widgets, chat UI, and business logic through helpers like `state`, `ensureSession()`, `hydrateUserFromProfile()`, `syncAuthStatus()`, `sendMessage()`, and `queueSqlLog()`—mirror `chat/docs/chat_architecture.md` whenever altering behavior.
- `/chat_api.php` is the PHP logger/proxy: `mode:"log"` persists messages into Postgres and `chat_transactions.log`, `mode:"proxy"` optionally relays to n8n; keep journal events and metadata keys stable.

## Messaging & Data Contracts
- Browser → n8n contract (см. `chat/docs/chat-n8n-json.md`) фиксирует оба поля: `session_id` (user-facing идентификатор, который видит и контролирует пользователь) и `chat_id` (серверный UUID для SQL). Не удаляйте `session_id` из payload.
- `queueSqlLog()` fires after every user + assistant turn; ensure it keeps parity with `sendMessage()` payloads so DB history and live replies never diverge.
- When adding webhook fields or response parsing, also update `handleAgentResponse()` plus the docs above so n8n nodes and QA stay in sync.

## Auth & Billing Surfaces
- `/auth/*` pages under `auth/` and Node service `/srv/gtc-auth/server.js` share helpers: always send `fetch` calls with `credentials:'include'`, expect the `gtc_post_auth` cookie, and honor envs such as `APP_BASE_URL`, `PAYMENT_PORTAL_URL`, `AUTH_SESSION_*`, and SMTP settings described in `mail.js`.
- Sidebar auth UI inside `chat/index.html` mirrors the legacy `/auth/` landing page (`chat/docs/Untitled-1.ini`); when adding new flows, reuse `postJson()` patterns and keep `goToFinish()` / `/auth/finish` redirects untouched.
- OTP blocks are feature-gated—`chat/docs/Untitled-2.ini` notes that `/auth/otp/*` may be disabled in prod. Hide or no-op UI wiring if the backend route is absent.
- Billing buttons and paywall handling must match `docs/billing/stripe-flow.md` and PHP portal `payment.php`, which bridge Stripe Pricing Tables via the `attach-metadata` AJAX hook.

## Persistence & Ops
- Database writes требуют валидных `gtc_user_id` и `chat_id`; если `chat_id` ещё не присвоен, `chat_api.php` создаёт его и возвращает в ответе. Новые изменения должны сохранять эту модель (пользователь управляет только `session_id`).
- Any schema or logging change needs parallel updates to `docs/chat_sql_persistence.md`, `chat/docs/chat_architecture.md`, and the migration scripts to keep ops playbooks accurate.
- Debug chat issues by tailing `/var/www/gtc-form/chat_transactions.log`; events like `log_mode_request`, `sql_insert`, `webhook_forward`, and `webhook_error` must remain trustworthy.

## Tooling & Testing
- Install dependencies with `npm install` (repo root) and `npx playwright install`. Execute UI regression via `npm run test:chat` (headless) or `npx playwright test tests/chat-menu.spec.ts --debug` when iterating.
- Tests assume `PLAYWRIGHT_TEST_BASE_URL=https://app.gtstor.com`; override the env if you spin up staging, and keep DOM ids/classes in sync with `tests/chat-menu.spec.ts` and `playwright-report/` artifacts.
- When touching auth flows, manually exercise email, Google GIS (`#googleBtnHost`), OTP (if enabled), resend verification, and `/auth/finish` paywall routing before merging—document outcomes in `chat/docs/chat_architecture.md` test log.

## Contribution Checklist
- Favor ASCII characters and inline assets; avoid bundlers or new runtime deps unless the page already includes them.
- Update related docs (`chat/docs/*.md`, `docs/chat_sql_persistence.md`, `docs/billing/stripe-flow.md`) whenever payloads, states, or user-visible flows change.
- Keep `safeNext()` redirect helper usage intact when adding new links/CTAs to prevent open-redirect regressions.
- Never invent IDs or accept chat traffic without `gtc_user_id`; defer to backend validation if unsure and document any interim shims in the relevant doc set.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kfilipenko)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/kfilipenko)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
