## Quick orientation

This repository is a Next.js (app router) frontend that uses EdgeOne-style edge functions (in `functions/api/*`) as API proxies to multiple AI providers. The UI is in `app/` and `components/`; server-side request routing and streaming proxies live in `functions/api/ai/index.js` and available-model discovery is in `functions/api/models/index.js`.

Keep guidance concise and concrete: reference these files when changing behavior, adding models, or adjusting stream parsing.

## Big-picture architecture (what to know first)
- Frontend: `app/` (React + server/client components). `app/page.jsx` is the main client page (has `use client`). `app/aiApi.js` implements the client-side stream parsing and is the canonical place for stream parsing logic.
- Components: `components/ChatHistory.jsx`, `ChatInputBar.jsx`, `MessageItem.jsx`, `ModelSelector.jsx` — these are where UI message shapes and streaming flags are expected.
- Edge Functions (server): `functions/api/ai/index.js` — model routing + provider proxies (DeepSeek, OpenAI, Gemini, Claude, Nebius). `functions/api/models/index.js` returns available models based on environment variables.
- Data flow: UI sends POST /api/ai with { model, messages } -> edge function proxies to provider -> provider streams SSE/Chunked JSON -> frontend `fetchAIStreamResponse` consumes stream and updates message objects.

## Important files to reference when coding
- Stream parsing and client behavior: `app/aiApi.js` (parseReasoningAndContent, fetchAIStreamResponse, onChunk handling)
- Proxy routing and provider adapters: `functions/api/ai/index.js` (proxyDeepSeek, proxyOpenAI, proxyGemini, proxyNebius, proxyClaude, plus streamProxy)
- Model availability: `functions/api/models/index.js` (decides which models are shown based on env vars)
- UI rendering patterns for messages: `components/MessageItem.jsx` (reasoning shown above answer; loading/streaming flags influence UI), `components/ChatHistory.jsx`, `components/ChatInputBar.jsx`, `components/ModelSelector.jsx`.

## Message shape & UI conventions (strictly used in this project)
- Messages are plain objects with at least: { role: 'user'|'assistant'|'system', content: string }
- Additional runtime flags the frontend uses (do not remove or rename without updating UI):
  - loading: boolean (temporary 'assistant is thinking' placeholder)
  - streaming: boolean (true while stream in progress)
  - reasoning: string (analysis/process text; MessageItem renders it above content)
  - streamingReasoning, streamingContent: booleans used by MessageItem to render cursor/animation states

When adding features that change message shape, update `MessageItem.jsx`, `ChatHistory.jsx` and the stream parsing helpers in `app/aiApi.js` together.

## Provider integration checklist (how to add a new model/provider)
1. Add provider API key to README and ensure env var is read by EdgeOne/hosting platform.
2. Add proxy function in `functions/api/ai/index.js` (follow existing proxies for streaming). Ensure you return `streamProxy(res)` (the frontend expects streamed body).
3. Add the model key in the main routing switch in `functions/api/ai/index.js` so `/api/ai` accepts the model string.
4. Optionally add the model to `functions/api/models/index.js` so it appears in the `ModelSelector` (it will be shown when its env var exists).
5. Test locally: `npm install` then `npm run dev` for frontend. To run functions locally use EdgeOne CLI as documented in README (`edgeone pages dev`) so the local `/api/*` endpoints are available.

## Local dev & debugging notes
- Frontend dev: `npm install` then `npm run dev`.
- Edge functions local dev: use EdgeOne CLI per README (`npm i -g edgeone`, then `edgeone pages dev`) — the README has the step-by-step.
- Common issues:
  - Missing env key -> `functions/api/models` will fall back to DeepSeek defaults. Check `functions/api/ai/index.js` for thrown errors like `API_KEY not set`.
  - Unknown model string -> `/api/ai` responds with { error: 'Unknown model' } (search `Unknown model` in `functions/api/ai/index.js`).
  - Stream format mismatch -> `app/aiApi.js` expects newline-separated `data: ` JSON chunks (SSE-like) and handles several provider formats (OpenAI, Claude, Gemini, DeepSeek Reasoner). If you introduce a new provider, add parsing logic to `fetchAIStreamResponse`.

## Examples to copy/paste
- Adding a new model to the model list (server-side):
  - Edit `functions/api/models/index.js` and push an object like { value: "your-model-key", label: "Your Model", disabled: false } when its env var is present.
- How the frontend calls the API:
  - POST /api/ai with JSON body { model: 'deepseek-chat', messages: [{role:'user', content:'...'}] }
  - The frontend expects streamed chunks and updates messages via the `onChunk` callback in `app/aiApi.js`.

## Small rules for AI agents editing this repo
- If updating stream parsing, include at least one unit test or a manual test note in the commit message describing which provider/format was validated.
- Avoid changing message field names. If rename is necessary, update all components that deserialize messages and the API proxies.
- When adding new env vars / provider secrets, update README's Required Environment Variables section.

If anything above is unclear or you want more examples (e.g. a small test harness for streaming JSON chunks), tell me which area to expand and I will iterate.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Alnold8) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
