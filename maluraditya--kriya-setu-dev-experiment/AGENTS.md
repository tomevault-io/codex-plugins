# Copilot Instructions for Excellent Academy

- Root entry: `App.tsx` controls navigation state (`DASHBOARD` / `TOPIC_VIEW`), active grade/subject/topic, and all topic-level lab component wiring.
- Topic config data: `data/index.ts`, `data/grades/11th.ts`, `data/grades/12th.ts`. Each `Topic.id` used in `App.tsx` and dashboard filtering.
- Visual flow: `Dashboard` -> topic card click -> `TopicLayoutContainer` -> topic-specific simulation component (e.g. `components/grade-12/physics/PhotoelectricLab.tsx`).
- No router library; routing is internal React state and conditional render in `App.tsx`.
- Most animations and layout use utility-first Tailwind-style classes in JSX; exceptions in `globals.css` for global styles.
- Data flow: topic selection loads `Topic` metadata (title, description, branch, unit, tour anchors), then shows `TextbookContent` alongside interactive lab in `TopicLayoutContainer`.
- Simulation components are mostly self-contained state + hooks (canvas, `setInterval` physics loops, chart updates). For cross-topic shared states (e.g., kinetics `SimulationConfig`), state is lifted to `App.tsx`.
- AI tutor path: browser calls `services/geminiService.ts` -> `/api/chat` (serverless handler). `api/chat.ts` posts prompt to Gemini endpoint; API key via `process.env.GEMINI_API_KEY`.

## Dev workflow
- install: `npm install`
- dev server: `npm run dev` (Vite 3000, host 0.0.0.0, PWA plugin enabled)
- prod build: `npm run build`, preview: `npm run preview`
- No test scripts currently in `package.json`.

## Key patterns and conventions
- Add new topic:
  1. add entry in `data/grades/{11th|12th}.ts` with unique `id`.
  2. create component under `components/grade-*/{subject}/`.
  3. import in `App.tsx` and add `case activeTopicId === '...'` in the selection branch (already structured with explicit imports for each lab).
- `types.ts`: central source of truth for `Topic`, `SimulationConfig`, `Screen`, `Subject`, `Grade`. Keep in sync with all components.
- Theme and icon mapping in `Dashboard.tsx` and on-topic UI uses `lucide-react` icons.
- Build output is in `dist`; PWA asset caching done via `vite.config.ts` with `vite-plugin-pwa`.

## Integration points
- External API: Gemini generative language in `api/chat.ts` (`gemini-2.0-flash`), environment variable required in deployment.
- UI state tracking: `activeTopicId` and `currentScreen` are central keys that drive which simulation and `aiContext` text is generated in App.
- Clouds: no backend DB; all curriculum content is in source code (topic array objects).

## Debugging notes
- For topic mismatch debug, verify `data/grades/*` `Topic.id` matches the string used in `App.tsx` `handleTopicSelect` and `TopicLayoutContainer` rendering conditions.
- To debug AI message flow, trace from `components/Assistant.tsx` -> `services/geminiService.ts` -> `api/chat.ts` and check network logs for 500 response from Gemini or missing `GEMINI_API_KEY`.

## Avoid
- Do not introduce client-side secrets. Serverless `/api/chat` holds the key.
- Keep React state local when possible; this app is intentionally stateful per-page without global state managers.

---
> Source: [maluraditya/Kriya-Setu-Dev-Experiment](https://github.com/maluraditya/Kriya-Setu-Dev-Experiment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-17 -->
