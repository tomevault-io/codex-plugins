## Quick orientation for AI code agents

This file captures the minimal, high-value knowledge an AI coding agent needs to be productive in this repository.

High-level architecture
- Single Node/Express server (root `server.js`) that serves both the frontend static bundle and two API surfaces:
  - `/api/ui/process` — used by the browser UI (no API_KEY required). Rate-limited via an in-memory limiter in `server.js`.
  - `/api/v1/process` — authenticated endpoint that requires `API_KEY` header (server enforces via `apiKeyAuth`).
- Backend image processing lives in `api/imageProcessor.js` (Jimp-based). Core function: `splitImageApi(imageUrl, targetChunkHeight, resizeWidth)` — it returns base64 data URIs for each chunk.
- Frontend TypeScript wrapper in `services/imageProcessor.ts` (`splitImage`) calls `/api/ui/process`, converts base64 into Blob/URL and returns a `ProcessingResult` (see `types.ts`).

Important files to inspect
- `server.js` — middleware, rate limiter, API auth, static serving of `bundle.js` and `output.css`, catch-all route for SPA.
- `api/imageProcessor.js` — main backend algorithm (Jimp). If you change output shape, update `services/imageProcessor.ts` accordingly.
- `services/imageProcessor.ts` — frontend fetch wrapper and base64 -> Blob conversion; used by `App.tsx` and `InputPanel.tsx`.
- `components/InputPanel.tsx`, `components/ResultViewer.tsx` — primary UI flows (form -> call -> render chunks / ZIP using JSZip).
- `types.ts` — canonical types: `ProcessedChunk`, `ProcessingResult`, `ProcessStatus`.

Build / dev / run workflows (explicit)
- Install deps: `npm install`
- Build + start (single command used by repo): `npm start`
  - `npm start` runs the `build` script then `node server.js`.
  - `build` runs: `tailwindcss -i ./input.css -o ./output.css` and `esbuild index.tsx --bundle --outfile=bundle.js` (produces `bundle.js` and `output.css` in repo root).
- Port: default `3001` (read `PORT` env); UI available at `http://localhost:3001` after `npm start`.
- Environment variables:
  - `API_KEY` (required for `/api/v1/process` — must be set in `.env` for server startup to expose API key behavior)
  - `RATE_LIMIT_PER_MINUTE` (optional, default 10)

Project-specific patterns & gotchas
- Server-side processing: the app intentionally avoids CORS by fetching and processing images on the server (see README and `api/imageProcessor.js`). When changing the backend, keep the same JSON response shape (base64 chunks) unless you update the frontend conversion code.
- Two API shapes for the same logic: `/api/ui/process` (used by UI) calls `splitImageApi` internally without client-supplied API key; `/api/v1/process` is the same logic but behind `apiKeyAuth`. Keep parity between them.
- Dynamic Tailwind class names: the UI composes classes like `text-${themeColor}-500`. Be careful about Tailwind purging — if you add new theme names or classes, ensure the build won't prune them (add safelist if necessary).
- Frontend blob handling: `services/imageProcessor.ts` converts base64 to Blob and creates `URL.createObjectURL(blob)`. If you change backend encoding (e.g., return raw buffers), update this conversion.

Examples (concrete snippets)
- UI request payload (from `services/imageProcessor.ts`):
  {
    "url": "https://example.com/large.jpg",
    "chunkHeight": 800,
    "resizeWidth": 1280
  }
- Backend core call (server): `const result = await splitImageApi(url, height, width);` (see `server.js` routes and `api/imageProcessor.js`).

Where to make common changes
- Adjust chunking/resizing algorithm: edit `api/imageProcessor.js` and run `npm start` to rebuild and test the UI flow.
- Add/modify authentication or rate limits: edit `server.js` (look for `apiKeyAuth` and the in-memory `rateLimitMap`).
- Frontend UX or behavior: `components/*` and `services/imageProcessor.ts` — remember to rebuild (npm start) to refresh `bundle.js`.

Testing & verification tips
- Quick manual test: `npm start`, open `http://localhost:3001`, paste an image URL and click `Process Image`. Watch console logs from `server.js` for backend messages.
- To exercise the authenticated API: set `API_KEY` in a `.env` and call `POST /api/v1/process` with header `API_KEY: <value>`.

If you intend to change response shapes
- Update `api/imageProcessor.js` output, then update `services/imageProcessor.ts` and `types.ts` simultaneously. Run the UI path (`InputPanel` -> `Process`) to verify conversion and downloads still work.

Notes for the agent
- Prefer small, focused PRs that touch server and client together when changing API contracts.
- Preserve the existing JSON result contract unless explicitly migrating (chunks: array of {id, base64, height, yOffset}).
- Keep `bundle.js` and `output.css` generation in mind — changes to TSX or tailwind require the build step used by `npm start`.

If anything above is unclear or you want me to include more examples (handlers, exact function signatures, or a safelist for Tailwind), tell me which area to expand.

---
> Source: [ddm21/split-stream-image-cropper](https://github.com/ddm21/split-stream-image-cropper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-10 -->
