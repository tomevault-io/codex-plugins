# Copilot / AI Agent Instructions — Sakhi Enterprises 2

Purpose
- Help AI coding agents become productive quickly by describing the app layout, run/debug commands, important patterns, and integration points.

Big picture
- This is a Node.js + Express backend serving a static frontend. Entry points:
  - `server.js` — top-level bootstrap: loads env, connects DB and mounts payment routes.
  - `backend/app.js` — creates the Express `app`, registers middleware and main routes.
- Static frontend files are served from `public/` and `product/` using `express.static`.
- MongoDB is used (see `backend/config/db.js`), and Razorpay integration is in `backend/config/razorpay.js`.

How to run (developer notes)
- Install dependencies: `npm install`.
- Ensure required environment variables in a `.env` file: at minimum `MONGO_URI`, `RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET`.
- Start the server: `node server.js` (server uses ES module `import` syntax; if Node complains, add `"type": "module"` to `package.json` or run with an ESM-capable launcher).

Where to make changes
- Routes: `backend/routes/*.js` (examples: `authRoutes.js`, `cartRoutes.js`, `orderRoutes.js`).
- Controllers: `backend/controllers/*Controller.js` implement request logic and should be updated for new endpoints.
- Models: `backend/models/*.js` define Mongoose schemas (look at `User.js`, `Order.js`, `cart.js`).
- Config: `backend/config/db.js` (DB connect) and `backend/config/razorpay.js` (payment setup).

Project-specific conventions
- App creation order matters: `backend/app.js` creates `app` first, then registers middleware, static assets, and finally routes. Follow the same order when adding middleware.
- Static content is served via absolute paths using `process.cwd()`; prefer `path.join(process.cwd(), ...)` when referencing files.
- Controllers follow a simple export pattern; prefer `export const handler = async (req,res) => {}` or default exports consistent with existing files.

Integration and environment notes
- DB: `connectDB()` in `server.js` loads `MONGO_URI` from env. Handle connection errors similarly (process.exit on fail).
- Payments: use `getRazorpayInstance()` from `backend/config/razorpay.js`. It expects `RAZORPAY_KEY_ID` and `RAZORPAY_KEY_SECRET` in env.

Small examples (how to add a route)
1. Add route file `backend/routes/newRoute.js` that `import express from 'express'` and `export default router`.
2. Implement logic in `backend/controllers/newController.js`.
3. Register in `backend/app.js` with `import newRoute from './routes/newRoute.js';` and `app.use(newRoute);` (after the `app` is created).

Common tasks an AI may be asked to do
- Add or modify API endpoints: update `routes/` and `controllers/`, adjust `models/` if persistence changes.
- Add new frontend static pages: place under `public/` or `product/` and ensure routes point to them if needed.
- Fix runtime ESM issues: add `"type": "module"` to `package.json` or convert `import` to `require` consistently.

Useful files to inspect first
- `server.js`, `backend/app.js`, `backend/config/db.js`, `backend/config/razorpay.js`, `backend/routes/*`, `backend/controllers/*`, `public/`, `product/`.

If anything is unclear
- Ask the repo owner to confirm preferred module system (CommonJS vs ESM) and to provide a `.env.sample` with required env vars.

---
Please review and tell me which area you'd like me to work on next (fix run scripts, implement an endpoint, or add a README). 

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dokeprerana2007)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/dokeprerana2007)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
