# GitHub Copilot Instructions

Purpose: short, actionable rules to make AI coding agents productive in this repo.

## Quick Reference

- **Single Source of Truth:** `app/hooks/useRegistry.ts`. Read/update registry via exported hooks (`agents`, `members`, `updateAgents(...)`, etc.). Never create a parallel source of truth.
- **Tier Check:** `currentUser?.tier === 'agent'`
- **Self-Propelled:** `agent.userId === currentUser?.id`
- **Audio:** PCM Int16 (Float32 × 32768)
- **Models:** `gemini-3-flash-preview` (text), `gemini-2.5-flash-native-audio-preview-12-2025` (voice)

## MCP Protocol

```
on_input:  breathe → see state
on_output: act store_event (if significance > 0.7)
```

AI is stateless. MCP is the memory.

## Key Files & Patterns

- `app/hooks/useRegistry.ts` — central state: `agents`, `teams`, `members`, `assets`, `history`, and derived `intelligence`. Use `updateAgents`, `updateMembers`, `updateTeams`, `updateAssets`, `updateHistory` to persist (localStorage keys defined in file).
- `app/services/geminiService.ts` — AI integration. Prefer `responseSchema` when expecting structured JSON (see `getPersonalizedTraining`, `translatePipelineData`).
- `app/types.ts` — canonical types for domain models.
- `packages/utils/pcm-audio.ts` — audio helpers: `encode`/`decode`, `float32ToInt16`, `createPCMBlob`, `decodeAudioData`, `createAudioStreamManager`, `playAudioChunk`.
- `supabase/functions/*` and `supabase/migrations/*` — Edge functions (Deno) and DB migrations. Functions expect environment variables and Supabase service role keys.

## Project Conventions (be prescriptive)

- Naming: `domain_topic_detail` for functions/files (consistent with repository style).
- No silent catches; surface or log failures like `console.error(...)`.
- No emojis unless requested; keep comments short and utilitarian.
- Never duplicate registry state in components — always use `useRegistry` as the single source.
- Self-propelled behavior: `currentUser?.tier === 'agent'` and `agent.userId === currentUser?.id` (used to gate interactions like drag/drop).

## Quick Examples

- Read registry: `const { agents, intelligence } = useRegistry();`
- Persist: `updateAgents(newAgents);`
- Gemini JSON response: set `responseMimeType: 'application/json'` and `responseSchema`.
- Play audio: `const mgr = createAudioStreamManager(); await playAudioChunk(mgr, base64Audio);`

## Testing

- **Unit/Integration:** `npm test` (Vitest + Testing Library + MSW)
- **E2E:** `npm run e2e` (Cypress) or `npm run test:e2e:ci` for CI
- **Typecheck:** `npm run typecheck`
- Test files in `app/tests/` — Gemini mocked via MSW, audio utils tested deterministically

## Operational Notes

- Run locally: `cd app && npm run dev`
- When editing Supabase functions, use provided env vars and test locally with Deno/Supabase CLI.
- Keep changes minimal and consistent with `instructions.md` functional doctrine and YAGNI.

---

If anything is unclear or you'd like a short example added (mutation, edge function test, or audio pipeline integration), tell me which part to expand.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mataluni-bravetto) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
