# AI Coding Assistant Instructions

## Quick snapshot
- Next.js 14 App Router statically exported for GitHub Pages; `next.config.js` sets `output: 'export'`, unoptimized images, and polyfills for Node built-ins.
- Root layout layers `LegacyAppWrapper` → `ErrorBoundary` → analytics/accessibility/cart/customer providers plus `ThemeScript`, install prompts, and manifest updaters—reuse this stack in any new layout.
- Supabase is the single source for menu, waiter calls, and AI endpoints; restaurant payloads are normalized into `Restaurant` from `src/components/data/index.ts`.

## Architecture & data flow
- Client restaurant routes (`src/app/restaurant/[slug]/`) fetch via `useRestaurantBySlug`, which chunks Supabase queries (`dish_categories`, complement junctions) and formats BRL prices; extend menu data inside that transformer rather than UI components.
- `RestaurantClientPage` owns view state (grid/list, tutorial overlays in `localStorage`, sorting via `SortModal`) and expects the fully populated `Restaurant`; lightweight rows from `useRestaurantList` only populate the switcher.
- Cart state lives in `src/context/CartContext.tsx` with 7-day persistence; calculate totals with `CartUtils.calculateUnitPrice` and generate IDs with `CartUtils.generateItemId` instead of writing new cart math.
- `CustomerDataProvider` and `CustomerCoordinatesProvider` persist to `localStorage`; call their update helpers instead of touching storage directly.
- Table identification via QR Code: `TableParamHandler` captures `?table=XX` params, saves to `localStorage` as `table_id`, and cleans the URL. Use `getTableId()` and `clearTableId()` from `src/components/TableParamHandler.tsx` to access/clear table data.

## Conversational & service flows
- Chatbot hooks (`src/hooks/useWebLLM.ts`) call the Supabase Edge Function `ai-chat` and fall back to `src/app/api/ai/chat/route.ts` (Gemini 1.5 Flash); they expect `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, and `GOOGLE_AI_API_KEY`. Preserve the PT-BR error messaging and recommendation extraction.
- Voice playback is handled by `useTextToSpeech`, which locks onto the Luciana voice and tracks opt-in state in `localStorage`; trigger speech through `speak()` and respect the `isEnabled` flag.
- Waiter call UX (`WaiterCallButton`, `WaiterCallNotifications`, `useWaiterCalls`) only renders when `restaurant.waiter_call_enabled` is true and posts to `/functions/v1/waiter-calls`; keep the bell audio + analytics hooks intact for parity with production.

## Build & test workflow
- Install with `npm install`; develop with `npm run dev`. `npm run build` runs `scripts/update-sw-cache.js` before `next build` and then touches `out/.nojekyll`—don’t bypass the prebuild or GitHub Pages will serve stale assets.
- Deploy static output through `npm run deploy` (publishes `out/` via `gh-pages`). Static export means API routes are dev-only; client fetches must handle missing env vars gracefully as in `useRestaurantBySlug` and `useWebLLM`.
- Playwright config (`playwright.config.cjs`) builds + starts the app unless `SKIP_WEBSERVER=1`; for targeted checks use scripts `test-gemma3.js`, `test-voice.js`, and `test-waiter-call.js`.

## Service worker & debugging
- `public/sw.js` skips `_next/static/**` caching and expires entries after ~1h; update cache logic through `scripts/update-sw-cache.js` and run `clear-all-caches.js` in the browser console when debugging stale data.
- Theme/accessibility preferences are synchronized via `AccessibilityProvider` + `ThemeScript`; when adding UI-level toggles, update both so PWA status bars stay accurate.
- Offline/legacy cleanup relies on `LegacyAppWrapper`, `ServiceWorkerCleanup`, and `useLegacyAppDetection`; reuse these helpers to avoid orphaned service workers when altering routing.

## References
- Layout & providers: `src/app/layout.tsx`
- Restaurant data + cart: `src/hooks/useRestaurantBySlug.ts`, `src/app/restaurant/[slug]/RestaurantClientPage.tsx`, `src/context/CartContext.tsx`
- AI & waiter flows: `src/hooks/useWebLLM.ts`, `src/components/ChatBot.tsx`, `src/hooks/useWaiterCalls.ts`
- Ops docs: `SETUP_ENVIRONMENT.md`, `DEPLOY_EDGE_FUNCTION.md`, `SERVICE_WORKER_CACHE_FIX.md`, `tests/README.md`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Yo-Self) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
