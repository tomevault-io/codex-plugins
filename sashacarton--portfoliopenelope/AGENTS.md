# Copilot instructions for PortfolioPenelope

Purpose: Give an AI coding agent the minimal, high-value knowledge to be productive quickly in this repo.

## Big picture (highly actionable) 🔧

- Monorepo split: **frontend/** (Vue 3 + TypeScript + Vite) and **backend/** (Strapi v5).
- Frontend is an SPA served in production via a Dockerized Nginx (see `frontend/Dockerfile` and `frontend/nginx.conf`).
- Backend is a Strapi application: content types live under `backend/src/api/<name>/content-types/<name>/schema.json`.
- Data flow: the frontend fetches Strapi REST endpoints (e.g. `https://api.penelopeletienne.ovh/api/projets?populate=Cover` in `frontend/src/views/Projects.vue`).

## Important developer workflows ✅

- Frontend (dev):
  - cd `frontend` && `npm install` and `npm run dev` → Vite dev server (default port 5173).
  - Type-check/build: `npm run build` runs `vue-tsc -b && vite build`.
  - Preview: `npm run preview`.
- Backend (dev):
  - cd `backend` && `npm install` and `npm run develop` → Strapi dev server.
  - Note: Strapi's port is **1338** by default in `backend/config/server.ts` (README mentions 1337—treat 1338 as authoritative).
- Docker/dev compose: `docker-compose.dev.yml` runs the frontend dev container (5173) and `docker-compose.yml` builds a production frontend image.
- Deploy: GitHub Actions runs `.github/workflows/deploy.yml` which SSHs to a VPS and executes `deploy.sh` on the server; `deploy.sh` fetches main and runs `npm install` + `npm run build` for the frontend only.

## Project-specific conventions & gotchas ⚠️

- Content types and API routes use French names (`projet`, `contact`) — look in `backend/src/api`.
- Frontend fetches production API URLs directly (see `Home.vue`, `Projects.vue`, `ProjectDetails.vue`). In dev, Vite proxy maps `/api` → `http://localhost:3000` (see `frontend/vite.config.ts`), which is a mismatch with Strapi's port (1338). When running locally, either:
  - change the proxy target in `vite.config.ts` to `http://localhost:1338`, or
  - run a small local forwarder on port 3000.
- Strapi database: by default uses **sqlite** at `.tmp/data.db` (see `backend/config/database.ts`), but it supports MySQL/Postgres via env vars.
- Authentication: `frontend/src/views/Login.vue` posts to `/api/login` (proxied). Admin token is saved in `localStorage` under `adminToken`.

## Patterns & code examples to reference 🧭

- Routes: `frontend/src/router/index.js` uses Vue Router with simple static imports.
- Projects fetch example: `frontend/src/views/Projects.vue` uses `fetch('https://api.penelopeletienne.ovh/api/projets?populate=Cover')` and expects JSON with populated `Cover` media.
- Strapi controllers/services: auto-generated in `backend/src/api/<name>/controllers` and `.../services` (use factories.createCoreController/service).
- 3D/VR page uses `three` in `frontend/src/views/Vr.vue`; follow existing usage rather than introducing a new 3D framework.

## Common edits you may be asked to do 🛠️

- Add a new Strapi content type: add `backend/src/api/<name>/content-types/<name>/schema.json` and a controller/service file if needed.
- Consume a new API field: update the frontend fetch to include `?populate=*` or specific relations, and adjust the view to use the new fields.
- Fix dev proxy issues: update `frontend/vite.config.ts` to point `/api` at Strapi's port if running locally on 1338.

## Environment & tooling 🧰

- Node/npm: backend requires Node >=18 and <=22 (see `backend/package.json` engines). Frontend uses modern Vite + TypeScript (`vue-tsc`).
- No unit tests or linters are present (none discovered); rely on manual testing and `vue-tsc` for type checks.

## Security / CI notes 🚨

- CI deploy uses SSH with `deploy.sh` and a repository secret `SAEZFRI_PASSWORD` via `sshpass` (see `.github/workflows/deploy.yml`); this is a plaintext-password based flow on the runner — consider using SSH keys only or safer secrets practices.

## Where to look next (key files) 🔎

- `frontend/` (Vite + Vue): `src/views/*`, `src/router/*`, `vite.config.ts`, `Dockerfile`, `nginx.conf`
- `backend/` (Strapi v5): `config/*.ts`, `src/api/*/content-types/*/schema.json`
- CI & deploy: `.github/workflows/deploy.yml`, `deploy.sh`

---

If anything in these notes is unclear or you want me to include extra examples (like a small dev checklist or a sample PR template), tell me which section to expand and I will iterate. ✨

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SashaCarton) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
