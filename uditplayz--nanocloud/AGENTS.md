<!-- Short, focused Copilot instructions for working in the Cloudrave repo -->
# Copilot instructions — Cloudrave

Keep suggestions targeted and actionable for this mini cloud-storage app. Refer to the files and patterns below when changing behavior or adding features.

1) Big-picture architecture
- Backend: `cloud-storage-backend` — an Express + Mongoose API that manages users and file metadata and issues S3 pre-signed URLs. Entry point referenced in `package.json` is `index.js`.
- Frontend: `cloud-storage-frontend` — Vite + React + TypeScript single-page app. UI components live in `components/` and rely on the TypeScript shapes in `types.ts`.
- Storage: Objects are stored in AWS S3. Backend creates pre-signed upload/download URLs and stores only metadata in MongoDB (`models/File.js`).

2) Critical flows (quote code examples when helpful)
- Authentication:
  - Backend uses JWTs. Middleware: `cloud-storage-backend/middleware/auth.js` expects the token in header `x-auth-token` (NOT `Authorization: Bearer`). Example: `req.header('x-auth-token')`.
  - Registration/login routes: `cloud-storage-backend/routes/auth.js` produce JWT signed with `process.env.JWT_SECRET` (expires in 3h).
- Upload flow (three-step):
  1. POST `/api/files/generate-upload-url` (backend) -> returns `{ uploadUrl, s3Key }`.
  2. Client PUTs the file bytes directly to `uploadUrl` (S3 presigned URL).
  3. Client POSTs `/api/files/finalize-upload` with `{ originalFilename, s3Key, mimetype, fileSize }` to save metadata in MongoDB.
  - See `cloud-storage-backend/routes/files.js` for exact payloads and S3 key format: `uploads/${userId}/${Date.now()}-${filename}`.

3) Environment & secrets
- Backend expects these env vars: `JWT_SECRET`, `AWS_REGION`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `S3_BUCKET_NAME` (used by `routes/files.js`).
- Frontend contains a demo Gemini integration in `cloud-storage-frontend/services/geminiService.ts` using `@google/genai` and `process.env.API_KEY` — do NOT commit real API keys into the frontend. Prefer moving secrets to the backend for production.

4) Dev / build commands
- Frontend: from `cloud-storage-frontend/package.json` use `npm run dev` (Vite), `npm run build`, and `npm run preview`.
- Backend: `cloud-storage-backend/package.json` has no dev script. The package lists `main: index.js` — search for `index.js` to run the API (e.g., `node index.js` or use `nodemon index.js` during development). Ensure env vars are set before starting.

5) Project-specific conventions & types
- The frontend uses `types.ts` to define `FileItem`, `User`, and `FileType`. Keep UI state consistent with these shapes (see `constants.ts` for `MOCK_FILES` examples).
- File metadata keys: `originalFilename`, `s3Key`, `mimetype`, `fileSize`, `owner`. Use these exact field names when interacting with backend endpoints.
- Auth header: always use `x-auth-token` for requests to protected endpoints.

6) When editing or adding features, check these files first
- Backend routes: `cloud-storage-backend/routes/files.js`, `cloud-storage-backend/routes/auth.js`
- Backend models: `cloud-storage-backend/models/File.js`, `cloud-storage-backend/models/User.js`
- Frontend integration and calls: components that handle upload and preview live in `cloud-storage-frontend/components/` and may use `services/geminiService.ts`.

7) Safety and tests
- Avoid embedding real credentials in commits. If you need to add a quick demo key, mark it clearly and add guidance to move it to a secure backend.
- The repo currently has no tests. When adding behavior to critical flows (auth, upload, delete), prefer creating small manual integration checks and documenting them in the PR.

If anything above is unclear or you'd like examples merged from other docs, tell me which file or flow to expand and I will iterate.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/uditplayz)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/uditplayz)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
