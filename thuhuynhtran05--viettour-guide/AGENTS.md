<!-- Auto-generated guidance for AI coding agents. Keep concise and actionable. -->
# Copilot / AI Agent Instructions for this repository

This file contains short, actionable guidance for AI coding agents working in this codebase. Focus on concrete, discoverable patterns and commands so contributors can be productive immediately.

1. Big picture
- Backend: Node.js + Express (CommonJS). Main backend code lives in `backend/src`. The HTTP server and Socket.IO setup are in `backend/src/server.js` while app routes and middlewares are assembled in `backend/src/app.js`.
- Frontend: static HTML/CSS/JS files in `frontend/` (no build step). Serve them directly or open files in browser / Live Server.
- DB: MongoDB (Mongoose models in `backend/src/models`). Data fixtures live in `data/` and a seeder script is available via `npm run seed`.

2. How to run & debug
- Install backend deps: `cd backend && npm install`.
- Run dev server (auto-reload): `npm run dev` (uses `nodemon src/server.js`).
- Run production server: `npm start`.
- Seed DB: `npm run seed` (seeder script located at `backend/src/seeder.js` if present).
- Env vars: create `backend/.env` with `PORT`, `MONGODB_URI`, `JWT_SECRET` (see `README.md`).

3. Key files & patterns to reference
- `backend/src/server.js` — HTTP server + Socket.IO. Socket events: `joinRoom` and `chatMessage`. Chat messages are persisted with `ChatMessage` model and broadcast via `io`.
- `backend/src/app.js` — Express app wiring. Routes mounted under `/api/*` (examples: `/api/auth`, `/api/locations`, `/api/guides`, etc.).
- `backend/src/routes/*.js` — route definitions; they import controllers from `backend/src/controllers`.
- `backend/src/controllers/*Controller.js` — per-resource controller logic. Follow existing naming and export patterns (CommonJS `module.exports`).
- `backend/src/models/*.js` — Mongoose schemas; file names are singular PascalCase (e.g., `ChatMessage.js`, `User.js`).
- `backend/src/middleware/errorHandler.js` — centralized error handling; use `next(err)` to surface errors.
- `backend/src/config/db.js` — DB connection logic; ensure `connectDB()` is called before starting server.
- `uploads/` and `app.use('/uploads', express.static(...))` — multer file uploads saved in `uploads` and served statically via `/uploads`.

4. Coding conventions & small rules
- Module system: CommonJS (`require` / `module.exports`). Do not convert files to ESM unless you update `package.json` type and all imports.
- Route/controller pattern: add a file in `routes/` and a matching controller in `controllers/`. Keep route handlers thin — implement business logic in controllers or services.
- Error handling: throw or `next(err)` from controllers; let `middleware/errorHandler.js` format responses.
- Socket.IO: writing to DB happens in `server.js` for chat; prefer the same pattern for room-based events—persist first, then `io.to(room).emit(...)`.

5. Integration points & dependencies
- MongoDB via Mongoose (`mongoose` in `package.json`). Connection string from env `MONGODB_URI`.
- Authentication: JWT (`jsonwebtoken`) — token handling done in `middleware/authMiddleware.js` and `controllers/authController.js`.
- File uploads: `multer` configured in `backend/src/config/upload.js`; uploaded files stored under `uploads/` and exposed at `/uploads`.
- Real-time: `socket.io` used directly in `server.js`. Messages saved to `ChatMessage` model.

6. When adding features
- Add Mongoose model in `backend/src/models` (PascalCase file name). Update any needed routes and controllers.
- Register new route in `backend/src/app.js` with `/api/<resource>` prefix.
- If server-side state is involved for real-time features, follow `server.js` pattern: persist, emit to room, then broadcast notifications if needed.

7. Scripts & maintenance
- `backend/package.json` scripts: `dev` (nodemon), `start` (node), `seed` (seed data). Use them as-is for local dev.
- No automated test harness discovered — add tests under a `tests/` folder if required; keep CommonJS style in tests to match project.

8. Examples (copy-paste patterns)
- Mounting a route: `const fooRoutes = require('./routes/foo'); app.use('/api/foo', fooRoutes);`
- Emitting chat message (persist then emit): see `server.js` chat handler — save `new ChatMessage(...)` then `io.to(room).emit('chatMessage', {...})`.

9. What an AI agent should not change without confirmation
- Do not convert CommonJS to ESM across multiple files.
- Do not change DB connection startup ordering (connect before listen) without validating the sequence.

If anything below is unclear or you want more detail for a specific area (seeder, auth, or upload flow), tell me which part and I will expand the instructions or add examples.

---
> Source: [thuhuynhtran05/VietTour-Guide](https://github.com/thuhuynhtran05/VietTour-Guide) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-30 -->
