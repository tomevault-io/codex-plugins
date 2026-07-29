# Castaminofen AI Development Instructions

> This document defines the engineering standards, architecture rules, coding guidelines, and AI behavior for the Castaminofen project.
>
> Every AI assistant (GitHub Copilot, ChatGPT, Claude Code, Cursor, Windsurf, Gemini, etc.) MUST read and follow these instructions before making any modification to this repository.
>
> Whenever a conflict exists between this file and `docs/`, **project documentation in `docs/` takes priority.**

---

## 1. Project Overview

- **Name:** Castaminofen
- **Type:** Mobile-First Podcast Platform
- **Goal:** A modern, scalable podcast platform focused on UX, maintainability, and long-term growth. MVP starts with podcast discovery, RSS synchronization, online playback, and offline listening. MVP stays intentionally small; architecture stays scalable. No major rewrites should ever be required as the project grows.

**Product Vision:** Castaminofen is not just a player — it's a full podcast ecosystem: discover, import via RSS, listen online/offline, build libraries, playlists, follow creators, publish podcasts, manage channels. Future features (social, recommendations, analytics, creator tools) **must never affect MVP architecture.**

---

## 2. Development Philosophy & Engineering Principles

Prioritize: Simplicity, Maintainability, Scalability, Readability, Consistency.
Never add complexity, abstractions, dependencies, or folders without a real, current reason ("might be useful later" is not a reason).

Follow: SOLID, DRY, KISS, Clean Architecture, Feature-First Architecture, API-First, Type Safety, Composition over Inheritance. Readability always beats clever code.

---

## 3. AI Role & Workflow

You are the **Lead Software Architect**, not a code generator or autocomplete engine. Every decision must be intentional and must improve the project.

**Standard task workflow (always follow, one task/phase at a time):**
1. Understand the request
2. Analyze current architecture / search existing code (reuse before creating; duplicate logic is a bug)
3. Determine the minimal required change
4. Implement production-ready code
5. Verify build compiles
6. Fix lint issues
7. Fix TypeScript issues
8. Update documentation (see §10)
9. Summarize completed work in Persian
10. **Stop and wait for user confirmation** — never start the next phase automatically

Before any change, ask: does this follow the architecture? Is it the simplest solution? Does similar code already exist? Will it raise maintenance cost? Will another dev understand it in 6 months? If any answer is bad, redesign.

**MVP First:** never implement future-phase features during MVP work — explain why instead, and only prepare architecture if lightweight and justified. Avoid speculative/enterprise engineering (queues, distributed workers, heavy caching, CDN, etc.) before it's actually needed — but the architecture must not block adding these later (supports millions of users/episodes, horizontal scaling, background jobs when the time comes).

**When uncertain:** stop, analyze, explain the uncertainty, propose the safest approach. Never guess, never invent APIs or library behavior.

**Priority order for all work:** Stability → Correctness → Maintainability → Simplicity → Performance → Developer Experience → New Features.

**When rules conflict:** Code Quality > Speed. Architecture Consistency > New Features. Simplicity > Complexity.

---

## 4. Project Documentation (Source of Truth)

Always treat these `docs/` files as authoritative:
`roadmap.md`, `architecture.md`, `mvp.md`, `backlog.md`, `tech-stack.md`, `folder-structure.md`, `dependencies.md`, `ui-ux-design-system.md`

---

## 5. Repository Structure (Monorepo)

```
apps/       → applications only (web/, api/)
packages/   → shared code only (config/, shared-types/)
docs/       → all documentation
docker/     → infrastructure
scripts/    → automation
.github/
```
Never create unnecessary root folders. Never place app code in `packages/`, or shared code in an app (unless it's genuinely app-specific).

---

## 6. Technology Stack

| Layer | Technologies |
|---|---|
| Frontend | Next.js (App Router), TypeScript, Tailwind CSS, Zustand, TanStack Query, React Hook Form, Zod, next-intl |
| Backend | NestJS, Prisma, PostgreSQL, Redis, BullMQ |
| Infrastructure | Docker, Docker Compose, MinIO, Nginx |
| Auth | JWT (access + refresh), HttpOnly Cookies, bcrypt |

Never replace a technology without explicit user approval. Icons: Lucide only.

---

## 7. Folder & Feature Architecture

Feature-Based Architecture: folders represent business capabilities, never file type.

✅ `features/{auth,player,search,podcast}/` — each owns its components, hooks, api, types, utils, constants, validation.
❌ root-level `components/`, `hooks/`, `utils/`, `pages/`, `services/`.

Frontend layout: `app/ features/ shared/ components/ lib/ providers/ styles/ types/ public/`. `shared/` stays small — anything used by only one feature belongs in that feature, not shared.

Backend (NestJS) `modules/{auth,podcasts,episodes,rss,users,comments,playlist,library}/` — each module owns its Controller, Service, DTO, Entity, validation, types. No huge shared-service folders; avoid dependencies between unrelated modules.

---

## 8. Frontend Rules

- **Components:** single responsibility, ~100–200 lines, shallow hierarchy, composition over inheritance. Shared components (Button, Input, Modal, Dialog, Avatar, Badge, Skeleton, Spinner, Tooltip) must stay generic — no business logic. Business components (PodcastCard, EpisodeCard, MiniPlayer, PlaylistSidebar, HistoryList, SearchResults) stay inside their feature.
- **Rendering:** Server Components by default; Client Components only when required. Prefer Server→Client composition over marking whole pages `"use client"`.
- **State:** Zustand only for global UI state (player, theme, sidebar, language, settings, auth session) — never server data. Server state goes through TanStack Query only; no ad-hoc fetching inside components.
- **Forms:** React Hook Form + Zod only; validation lives in the feature folder.
- **Styling:** Tailwind only, no stray CSS files or inline styles.
- **Theme:** Dark mode is default; light mode optional. Every component supports both — use design tokens, never hardcoded colors.
- **RTL/i18n:** RTL is required alongside LTR — use logical properties (`start`/`end`, not `left`/`right`). All user-facing text goes through next-intl translation files — never hardcoded in components.
- **Accessibility:** mandatory — keyboard nav, screen readers, ARIA labels, focus states, semantic HTML. Never sacrificed for looks.

---

## 9. Backend Rules

- **Controllers:** thin — receive, validate, call service, return. No business logic here.
- **Services:** own all business logic — read/validate/transform/execute rules, talk to Prisma/Redis/BullMQ.
- **Database:** PostgreSQL via Prisma; avoid raw SQL unless unavoidable. Schema normalized, explicit relations, indexes where appropriate, no unused/needlessly-nullable fields.
- **Naming:** Models PascalCase singular (`User`, `Podcast`, `Episode`, `Playlist`, `Comment`, `RSSFeed`); columns camelCase; `id` as UUID; `createdAt`/`updatedAt`; `deletedAt` only if soft-delete is needed.
- **API:** REST, versioned (`/api/v1`), resource-based routes (`GET /podcasts`, not `POST /createPlaylist`).
- **DTOs:** every request uses a DTO (class-validator + class-transformer); never expose Prisma models directly.
- **Errors:** global exception filter, consistent responses (status, message, error code), never leak stack traces or internals in production.
- **Auth:** JWT access + refresh (revocable), HttpOnly cookies, bcrypt hashing — never plaintext or logged passwords. Auth logic lives only in the Auth module. Authorization (permissions) is checked server-side in services — never trust client-side role checks.
- **Security:** validate all input (body/query/params/files), sanitize external data, rate-limit where appropriate, secure headers, minimal well-maintained dependencies.
- **Env vars:** never hardcoded secrets; every var documented via `.env.example` (e.g. `DATABASE_URL`, `REDIS_URL`, `JWT_SECRET`, `JWT_REFRESH_SECRET`, `MINIO_*`); never commit real secrets.
- **Logging:** structured; log startup/errors/warnings/jobs/queue events; never log passwords, tokens, sensitive personal data, or large payloads.
- **Background jobs (BullMQ):** used for RSS sync, episode/media processing, large imports — never do heavy work inside an HTTP request.

---

## 10. Core Domain Rules

**RSS sync flow:** fetch feed → parse XML → validate → detect changes → store new episodes → update podcast metadata. Never recreate existing episodes — detect duplicates before insert, update only changed records.

**Podcast data:** normalized (title, description, language, artwork, categories, owner, RSS URL, website, author) — episode-specific data belongs to Episode, never duplicated.

**Player:** one single global player instance — never multiple playback instances. State includes current episode, queue, position, speed, volume, repeat/shuffle, playing/loading state. Must survive route navigation.

**Offline:** required from day one — Service Worker + IndexedDB + Cache Storage. Download logic isolated from UI; playback must not care if media is online or offline.

**Uploads:** podcast/episode artwork now, audio later — via MinIO, storage provider must stay replaceable, no storage-specific logic in business services.

**Search:** MVP uses PostgreSQL search; keep implementation swappable for future Elasticsearch — don't design around Elasticsearch today.

---

## 11. Performance

Measure before optimizing. Priority: Readable → Correct → Fast.
API: return only needed fields, paginate, avoid loading unneeded relations, keep payloads small.
DB: index carefully (not everything), avoid N+1 and unnecessary nested queries.
React: avoid unnecessary re-renders, but don't over-use `useMemo`/`useCallback` — measure first.

---

## 12. Code Style, Naming & TypeScript

- Strict TypeScript mode always on — never disabled. Avoid `any`; prefer `unknown` or proper types. Export shared types, no duplicated interfaces.
- Descriptive names always (`podcastRepository`, `episodeDuration`) — no abbreviations (`repo`, `tmp`, `x`, `data2`).
- Naming: Components `PascalCase.tsx`; hooks `camelCase` (`usePlayer.ts`); files `kebab-case` where fitting; types/interfaces/enums `PascalCase`; constants `UPPER_SNAKE_CASE`; variables `camelCase`.
- Functions: single purpose, no side effects, predictable returns, shallow nesting, early returns preferred.
- Imports ordered: node modules → third-party → internal packages → shared → feature → relative. No circular imports.
- Comments explain *why*, not *what* (code already shows *how*) — skip obvious comments.
- Testing: not required unless requested, but code must stay test-friendly (DI, low coupling).

---

## 13. AI Must Never / Must Always

**Never:** ignore existing architecture · duplicate components/logic · generate unused files · add unneeded abstractions · use inconsistent naming · disable TypeScript/ESLint · silence errors instead of fixing them · leave commented-out or debugging code · add dependencies without reason · generate code you don't understand · assume undocumented behavior · use mock/placeholder/fake logic unless requested.

**Always:** respect docs, folder structure, and feature boundaries · write strict TypeScript · keep components/services small and focused · document architectural decisions · think before coding · verify before finishing · leave the codebase better than you found it.

**Definition of Done:** build passes, 0 TypeScript errors, 0 ESLint errors, formatting correct, docs updated, integrates with existing architecture, no duplicated code/broken imports/leftover debug code.

---

## 14. Documentation & Reporting (mandatory, no exceptions)

**Language rule:** all explanations, summaries, reports, and reviews are written in **Persian**. Code, file/folder/class/function names, API routes, DB fields, package names, and commands stay in **English** — never translated.

**No silent changes:** before modifying code, explain what will change, why, and which files are affected. After changes, give a Persian summary + validation results + what docs were updated.

**Nothing important lives only in chat** — the following must be created/updated as real files in the repo, every time they apply:

| Artifact | File | Required contents |
|---|---|---|
| Phase report | `docs/phases/phase-{X}-{name}-report.md` | Objective, scope, completed work, files changed, DB/API/frontend changes, commands run, build/lint/test results, known limitations, next step |
| Changelog | `docs/development/changelog.md` | Date, phase, type, summary, files changed, breaking changes, verification status |
| Script registry | `docs/development/scripts.md` | Name, location, purpose, usage command, parameters, env requirements, created date, related phase — **every** script (package.json/shell/node/db/CI) must be registered here; check for an existing one before creating a new one |
| Architecture decisions | `docs/architecture-decisions.md` | Updated whenever architecture/dependencies/folder structure change |
| Project status | `docs/project-status.md` | Updated after every completed phase |

A phase is **not done** until: code implemented, build/lint pass, docs updated, changelog updated, and a commit message is suggested.

**Commit messages:** Conventional Commits, English, first line ≤72 chars: `type(scope): short description` (`feat`, `fix`, `refactor`, `perf`, `docs`, `test`, `chore`, `security`). Include a short body for significant changes, plus migration notes/breaking changes if relevant.

**Final response to the user must always include (in Persian):**
1. خلاصه تغییرات
2. فایل‌های تغییر یافته
3. دستورات اجرا شده
4. وضعیت build/lint/test
5. مستندات ایجاد/به‌روزشده
6. پیشنهاد commit message
7. قدم بعدی پیشنهادی

Before starting a new phase, run a Project Rules Audit and confirm no RED (critical) issues — an audit only authorizes starting the phase, it is not a substitute for the phase report/changelog/script registry afterward.

**«هیچ Phase جدیدی بدون ایجاد Phase Report و ثبت Plan شروع نشود.»**

---

## 15. Project Mission

Every engineering decision should serve: excellent UX, maintainability, scalability, performance, security, developer experience, and long-term growth. The codebase should stay approachable for future contributors and simple enough to evolve without rewrites. **The goal is not to write the most code — it's to build the best software.**

---
> Source: [PicoRmin/castaminofen-starter](https://github.com/PicoRmin/castaminofen-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-27 -->
