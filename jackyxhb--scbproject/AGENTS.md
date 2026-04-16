# SCBProject — AI working guide

Make AI agents productive fast. Sources of truth: `README.md`, `scb.instructions.markdown`, `CLAUDE.md`.

## Big picture

- Product: Smart Courier Box (SCB) — secure delivery box with QR-based access; flows are time-bound and auditable.
- Architecture (FERN): React (Vite) frontend + Express/Node backend + Firebase (Auth/Firestore) + Vercel. Optional Raspberry Pi prototype.
- Core flows: register → courier unlock/deposit → backend issues one-time encrypted QR → recipient unlocks.

## Repo map

- Root: `vercel.json`, `package.json`, `.env.example`, `.vercelignore`.
- Frontend (`/frontend`): `src/api.js` (all fetches), `vite.config.js`; uses `VITE_API_BASE=/api`.
- Backend (`/backend`): `server.js` (Express app), routes `routes/api.js`, `routes/boxes.js`, `routes/parcels.js`.
  Libs: `lib/qr.js`, `lib/crypto.js`, `lib/store-*.js`, `lib/auth.js`, `lib/notify.js`, `lib/rateLimit.js`, `lib/db.js`.
  Tests: `backend/__tests__/*.test.js`. Scripts: `backend/scripts/print-public-key.js`, `backend/scripts/purge-expired.js`.

## Workflows

- Dev: `npm install` → `npm run dev` (Vite @5173, API @3000). Optional: `npm run dev:ui`, `npm run dev:api`.
- Tests: `npm -w backend test` (Jest+Supertest; ESM enabled via script). Add new tests under `backend/__tests__`.
- Build/Deploy: Vercel uses `npm run vercel-build` and `vercel.json` (routes `/api/*` → backend, SPA fallback → `frontend/dist/index.html`).
- CI: `.github/workflows/ci.yml` runs backend tests and builds both apps on PRs.

## Conventions & patterns

- API shape: JSON only `{ ok, data?, error? }`. Always `res.json(...)`. Mount under `/api` via routers.
- Security: RSA-OAEP-SHA256 for QR payloads (`lib/crypto.js`). Keys can persist to `.keys` via env; otherwise in-memory.
- One-time + TTL: `lib/qr.js` with memory or Firestore store (`lib/store-*.js`), single-use consumption and expiry enforced. Admin purge: `POST /api/admin/purge-expired` (token via `ADMIN_TASK_TOKEN`).
- Auth: `lib/auth.js` attaches Firebase user from `Authorization: Bearer <idToken>`; gate with `REQUIRE_AUTH=1`.
- Rate limiting: `lib/rateLimit.js` applied to sensitive routes (e.g., `/api/qr/generate`, `/api/qr/validate`, `/api/parcels/:id/notify`). Tune via `RATE_LIMIT_*` envs.
- CORS/headers: `helmet()` and `cors({ origin: CORS_ORIGIN || derived VERCEL_URL })`. On Vercel, server exports the app (no `app.listen`).

## Integration points

- Firebase Admin: configure via env (`FIREBASE_PROJECT_ID`, `FIREBASE_CLIENT_EMAIL`, `FIREBASE_PRIVATE_KEY`). Toggle Firestore with `USE_FIRESTORE=1`.
- Notifications: `lib/notify.js` uses nodemailer (SMTP*URL/SMTP_FROM) and optional Twilio (TWILIO*\*). Notify via `POST /api/parcels/:id/notify`.
- Frontend API: `frontend/src/api.js` wraps calls (`health`, `qr/public-key`, `qr/generate`, `qr/validate`, `parcels` CRUD/notify); UI panels live in `src/App.jsx`.

## Examples to mirror

- Endpoints: `POST /api/qr/generate`, `POST /api/qr/validate`, `GET /api/qr/public-key`, `POST /api/parcels`, `PATCH /api/boxes/:id/state`.
- Tests: see `backend/__tests__/qr.test.js` (single-use + expiry), `public-key.test.js`, `boxes-parcels.test.js`, `notify.test.js`.
- Scripts: `npm -w backend run key:print` (prints PEM), `npm -w backend run qr:purge`.

## Gotchas

- Wrong route target → HTML: ensure requests go to `/api/*` and endpoints always `res.json(...)`.
- CORS on Vercel: set `CORS_ORIGIN=https://your-domain` if needed; `VERCEL_URL` is host-only in prod.
- Frontend must use `VITE_API_BASE=/api` in prod; avoid absolute localhost URLs.

When adding features: keep the API shape, add rate limits where sensitive, thread secrets via env only, update `frontend/src/api.js`, and add focused Jest tests.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jackyxhb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
