# Studio Agents — AI Coding Agent Guide (studio-agents)

This is the **main development repo** for Studio Agents, an AI-powered music production platform with 16 specialized agents for lyrics, beats, vocals, video, and more. Backend on Railway, frontend on Vercel, Firebase for auth/data/storage.

## Architecture

```
frontend/          React 19 + Vite 7 (ES modules)
  src/App.jsx        Hash-based router, lazy-loads all views
  src/constants.js   BACKEND_URL auto-detection + AGENTS array (16 agent definitions)
  src/firebase.js    Firebase SDK init (Auth, Firestore, Storage) + upload/delete helpers
  src/components/    StudioView (main UI), LandingPage, StudioOrchestratorV2,
                     ProjectHubV3, AdminAnalytics, MultiAgentDemo, etc.
  src/hooks/         useSafeAsync, useVoice, useLazyLoadImages, useSwipeNavigation
  src/utils/         demoMode (code "pitch"), errorMonitoring, analytics, mediaUtils

backend/           Express 5 + CommonJS (Node 18+)
  server.js          Single ~7000-line file — all routes, middleware, AI integrations
  services/          emailService, beatDetectionService, videoCompositionService,
                     videoGenerationOrchestrator, audioMixingService, userPreferencesService
  .env               GEMINI_API_KEY, FIREBASE_*, ELEVENLABS_API_KEY, REPLICATE_API_*, STRIPE_*
```

## Deployment & URLs

- **Backend (Railway):** Dockerfile → `sh -c 'cd /app/backend && node server.js'` on port 3001
  - Production: `https://web-production-b5922.up.railway.app`
- **Frontend (Vercel):** `cd frontend && npm run build` → `frontend/dist`
  - `vercel.json` rewrites `/api/*` to the Railway backend
- **Firebase project:** `studioagents-app` — shared with the whip-montez-live repo

## Critical conventions

- **BACKEND_URL detection:** `frontend/src/constants.js` checks `window.location.hostname`/port. Localhost → `http://localhost:3001`, production → empty string (Vercel rewrites handle it).
- **Firebase config is in source code** (`frontend/src/firebase.js`). Do NOT move to env vars — it's intentional for client-side SDK. Backend uses Firebase Admin with service account from env vars.
- **Hash-based routing:** `#/`, `#/studio/{tab}`, `#/whitepapers`, `#/legal`. No react-router — uses `window.location.hash` + `hashchange` listener in `App.jsx`.
- **Agent definitions live in `constants.js`** as the `AGENTS` array (id, name, category, tier, capabilities, etc.). Adding an agent = adding an entry here + handling in `StudioView.jsx`.
- **server.js is one large file.** All routes, middleware, and integrations are co-located. Key sections are marked with `// ====` banners.

## Auth & credits

- Firebase Auth (Google, Email/Password, Apple Sign-In). Token verified backend-side via `verifyFirebaseToken` middleware.
- **Admin emails** hardcoded in `server.js` (`ADMIN_EMAILS` array). Admin middleware: `requireAdmin`.
- **Credit system:** `CREDIT_COSTS` object in `server.js` defines per-feature costs (text=1, beat=5, video=15, etc.). `checkCreditsFor(type)` middleware deducts credits via Firestore transactions. Admins bypass credit checks.
- **Tiers:** free (4 agents), monthly (8 agents), pro (all 16), lifetime.

## AI integrations (all in server.js)

| Service | Model | Route | Env var |
|---------|-------|-------|---------|
| Google Gemini | `gemini-2.0-flash` (fallback: 1.5-flash) | `POST /api/generate` | `GEMINI_API_KEY` |
| Replicate | Flux 1.1 Pro | `POST /api/generate-image` | `REPLICATE_API_TOKEN` |
| Replicate | MusicGen | `POST /api/generate-audio` | `REPLICATE_API_TOKEN` |
| Replicate | BARK | `POST /api/generate-speech` | `REPLICATE_API_TOKEN` |
| Stripe | Payments/subscriptions | `POST /api/stripe/*` | `STRIPE_SECRET_KEY` |

## Running locally

```powershell
# Backend (needs backend/.env with API keys)
cd backend && npm install && node server.js    # http://localhost:3001

# Frontend (separate terminal)
cd frontend && npm install && npm run dev      # http://localhost:5173
```

## Testing

```powershell
cd frontend && npx playwright test       # E2E tests
cd frontend && npx playwright test --ui  # Interactive test UI
cd frontend && npm run check:circular    # Check for circular dependencies
```

## Key patterns

- **Lazy loading:** All heavy components use `React.lazy()` in `App.jsx`
- **Demo mode:** Activated via code "pitch" or `?demo=pitch` — returns mock data, skips credits (`frontend/src/utils/demoMode.js`)
- **Middleware chain:** `verifyFirebaseToken → requireAdmin/checkCreditsFor → rateLimiter → handler`
- **Error handling:** Winston logger (file + console), Morgan for HTTP logs, `ErrorBoundary.jsx` on frontend
- **Vite build:** Chunk splitting (firebase, react, icons, vendor), gzip+brotli compression, terser minification
- **Offline persistence:** Firestore `enableIndexedDbPersistence` is enabled in `firebase.js` for resilience on flaky connections

## What NOT to do

- Do not commit secrets or full API keys. Backend logs only 8-char key substrings.
- Do not refactor `server.js` into multiple route files unless explicitly asked — the single-file pattern is intentional.
- Do not change Firebase config in `firebase.js` unless instructed. The Firebase project is live.
- Do not remove `ADMIN_EMAILS` or `DEMO_ACCOUNTS` constants — they're used for testing and demos.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Jari57) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
