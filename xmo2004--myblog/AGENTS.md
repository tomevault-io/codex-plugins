# Copilot Instructions
- Monorepo: React/Vite frontend in `client`, Express/Prisma (SQLite) backend in `server` with REST API under `/api`.
- Local dev shortcut: run `./start-dev.sh` (kills 3001/5173, starts backend via `npm start` and frontend via `npm run dev`).
- Backend entry: [server/src/app.js](server/src/app.js); uses `helmet`, CORS allowlist from `CORS_ORIGINS` (comma list, defaults to localhost 5173), global rate limiting, JSON body limit `JSON_BODY_LIMIT`.
- Auth: JWT in `Authorization: Bearer` with `JWT_SECRET`, issuer `blog`, audience `blog-client` ([server/src/middleware/auth.middleware.js](server/src/middleware/auth.middleware.js)). Use `verifyToken`/`verifyTokenOptional` plus `isAdmin`/`isEditorOrAdmin` to guard routes.
- Persistence: Prisma schema in [server/prisma/schema.prisma](server/prisma/schema.prisma) (SQLite by default `DATABASE_URL=file:./dev.db`). Models cover posts/tags/categories (3-level hierarchy), comments+likes, projects/resources, columns/column nodes, bookmarks, weight/diet tracking, operation/visit logs.
- Backup/ops: daily backup scheduled at start via `scheduleDaily` ([server/src/services/backup.service.js](server/src/services/backup.service.js)) using `BACKUP_SCHEDULE` (HH:MM) and `BACKUP_RETENTION_DAYS`. Admin endpoints expose backup/restore/list and logs.
- Logging: mutations call `logOperation` ([server/src/middleware/log.middleware.js](server/src/middleware/log.middleware.js)) to persist before/after snapshots with user/id/ip into `OperationLog`.
- Post access: `accessLevel` (`regular|plus|pro`) enforced in [server/src/controllers/post.controller.js](server/src/controllers/post.controller.js) when fetching a post; admins bypass, otherwise requires matching membership. Public list filters: `search`, `category|categoryId` (includes descendants), `tag`, `sort`.
- Post routes: [server/src/routes/post.routes.js](server/src/routes/post.routes.js) exposes public `GET /api/posts`/`/api/posts/:id`, admin `GET /api/posts/admin/all`, editor/admin create/update/delete, admin bulk delete; comments create requires auth, likes toggled via `/api/comments/:id/like`.
- Auth routes: [server/src/routes/auth.routes.js](server/src/routes/auth.routes.js) with rate-limited register/login/check/send-code plus `/me` get/update behind auth.
- Admin routes: [server/src/routes/admin.routes.js](server/src/routes/admin.routes.js) require admin token; stats/backup/restore/logs/self-update/user management. Role guard: only username `xmo2004` can become/remain admin ([server/src/controllers/admin.controller.js](server/src/controllers/admin.controller.js)).
- API client: axios wrapper with base `VITE_API_BASE_URL` defaulting to `http://localhost:3001/api`, attaches token from `localStorage`, 401 handler clears token + redirects to `/login` ([client/src/lib/api.js](client/src/lib/api.js)).
- Frontend routing: [client/src/App.jsx](client/src/App.jsx) uses `react-router-dom` with lazy-loaded pages; `MainLayout` for public pages, `AdminLayout` under `/dashboard` wrapped by `ProtectedRoute` (token presence only), `VisitTracker` logs non-dashboard hits to `/api/visit`.
- UI stack: Tailwind v4, TipTap editor, mermaid, KaTeX, recharts, three.js for background; CSS entry [client/src/index.css](client/src/index.css) and layouts/components under `client/src/components` and `client/src/layouts`.
- Tests: backend `npm test` runs `node create-admin.js` then `node --test`; ensure Prisma client generated (`npm run build` or `npx prisma generate`) and DB exists before running.
- Seeding/admin: `server/create-admin.js [username] [password]` upserts admin (default `xmo2004/admin123`).
- Backups directory: `server/backups`; Prisma DB file at `server/prisma/dev.db` by default; restore endpoint copies selected backup into that location.
- REST pattern: controllers use `global.prisma` singleton; prefer `include`/`select` to control response size; sanitize user-facing responses (e.g., post comments return `likeCount`/`isLiked` instead of raw likes).
- Rate limits: global limiter (`RATE_LIMIT_WINDOW_MINUTES`, `RATE_LIMIT_MAX` default 15 min/300) applied to all routes; `loginLimiter` (`LOGIN_RATE_LIMIT_*`) protects auth endpoints.
- Dev ports: backend 3001, frontend 5173; update CORS/`VITE_API_BASE_URL` together when changing.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/XMo2004)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/XMo2004)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
