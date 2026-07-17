cd c:\Users\andre\EduSys-Connect
npx vitest run tests/adminMessagesThreadView.test.tsx tests/adminMessagesThreadList.test.tsx tests/adminMessagesFilters.test.tsx tests/useAdminMessages.test.ts# Copilot / AI agent instructions for EduSys-Connect

Purpose: concise, actionable guidance for AI coding agents to be productive in this repo.

1) Big-picture architecture
- Frontend: React 19 + TypeScript (Vite) + Tailwind. Routes use `/app/{role}/{module}` with role-based guards.
- Backend: Supabase (Postgres + Auth). Normalized tables: `subjects`, `subject_classes`, `teacher_subjects`, `classes_consolidated`.
- Services: `services/supabaseDataService.ts` is the primary integration layer (CRUD, linking, migrations). `localStorageService.ts` provides offline/demo fallbacks.

2) Critical workflows & commands
- Start dev server: `npm run dev` (Vite). Port 3001 default.
- Run tests: `npm run test` or `npx vitest` (Vitest config in `vitest.config.ts`). Example tasks in `.vscode/tasks.json`.
- Seed CBC subjects: `node scripts/seed-cbc-subjects.ts` or use SQL variant `scripts/seed-cbc-subjects.sql` with `supabase db query -f`.

3) Project-specific conventions & patterns
- Separate-login per role: each portal stores its own session; do not assume a single shared user session. See `LOGIN_ARCHITECTURE_GUIDE.md`.
- Grade values can be either class `id` or human label; use `migrateSubjectsToClassIds()` and `classOptions` mapping to normalize.
- `subject_classes` join table maps subjects → classes (streams are suffixes derived from `classes_consolidated.name`).
- `localStorageService` is intentionally used for legacy/demo behavior; prefer server ops in `supabaseDataService` for authoritative changes.
- `is_read_only` flag in localStorage toggles guarded write behavior in services.

4) Integration points & secrets handling
- Edge functions under `supabase/functions/` require `SERVICE_ROLE_KEY` for privileged actions; calls must include `Authorization` header.
- Do not invent or mock real users/data — use existing DB records. See the repo rule: NO FAKE DATA.

5) Files to inspect first (high ROI)
- `services/supabaseDataService.ts` — look here for create/update flows, JWT refresh, and stream/linking logic.
- `services/subjectLinksService.ts` — helpers to fetch/set subject↔class links.
- `services/localStorageService.ts` — migration helpers, CBC initializer, and legacy CRUD used by UI.
- `components/Forms/SubjectForm.tsx` and `components/Academic/SubjectsView.tsx` — UI flows for creating and displaying subjects/streams.
- `scripts/seed-cbc-subjects.ts` and `scripts/seed-cbc-subjects.sql` — canonical seeding logic and examples of mapping streams to classes.

6) Testing notes
- Many tests assume Supabase stubs; prefer running small focused tests first (e.g., `tests/subjects.test.ts`).
- If tests hit DB schema issues (missing tables), run seed/migrations before broader test runs.

7) Agent rules (explicit)
- Follow repository 'NO FAKE DATA' guidance: never invent database records; if a record is required, report what's missing.
- Minimize changes; when editing services prefer updating `supabaseDataService.ts` and `subjectLinksService.ts` to keep behavior centralized.
- When adding UI changes, update `types.ts` and corresponding tests.

If anything here is unclear or you want more examples (e.g., a short walkthrough to add a subject with streams end‑to‑end), tell me which area to expand.
<parameter name="filePath">c:\Users\andre\EduSys-Connect\.github\copilot-instructions.md
---

# Output Control Rules

## Priority
Minimize token usage.
Return shortest correct answer.

## Format
Default to code only.
No introductions.
No summaries.
No restating prompt.
Max 3 short sentences if explanation is required.

## Reasoning
Do not show chain-of-thought.
Do not show analysis.
Do not justify decisions.

## Debugging
State issue in one sentence.
Return fixed code only.

## Length Limit
Keep under 150 words unless explicitly asked for more.

---
> Source: [Rhinoheart/EduSys-Connect](https://github.com/Rhinoheart/EduSys-Connect) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-17 -->
