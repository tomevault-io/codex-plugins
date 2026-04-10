# Copilot Instructions for Habicht (UniSports)

## Project Overview
- **Habicht** is a Next.js 14/React/TypeScript platform for Swiss volleyball scouting, with player/recruiter roles, club integration, and video uploads.
- Key directories: `app/` (Next.js routes/pages), `components/` (UI, player, shared), `lib/` (helpers, Prisma), `prisma/` (schema, migrations), `scripts/` (data import/export), `data/` (static JSON), `contexts/` (React contexts).

## Architecture & Data Flow
- **Frontend**: Next.js app directory structure, with dynamic routes (e.g., `app/players/[id]/page.tsx`).
- **Backend**: Next.js API routes in `app/api/` (e.g., `app/api/players/`, `app/api/videos/`).
- **Database**: PostgreSQL via Prisma (`prisma/schema.prisma`).
- **Auth**: NextAuth.js, see `app/api/auth/` and `types/next-auth.d.ts`.
- **File Uploads**: Cloudinary integration for images/videos.
- **Swiss Club Integration**: Club data and linking logic in `lib/clubsDatabase.ts`, `lib/clubMatcher.ts`, and related files.

## Developer Workflows
- **Install**: `npm install`
- **Dev server**: `npm run dev` (Next.js)
- **Build**: `npm run build`
- **DB push**: `npm run db:push` (Prisma)
- **Prisma Studio**: `npm run db:studio`
- **Scripts**: Run with `npx ts-node scripts/<script>.ts` (see `scripts/README.md` for scraping/importing player data)
- **Deploy**: Vercel (see `VERCEL_SETUP.md`)

## Project Conventions
- **TypeScript**: All business logic and components are typed; extend types in `types/` as needed.
- **Tailwind CSS**: Used for all styling; utility classes preferred.
- **React Context**: Language/theme/context in `contexts/`.
- **API**: Use axios/fetch for client-server communication; see `lib/` for helpers.
- **Swiss-specific logic**: Canton/club/league logic in `lib/` and `data/`.
- **CV Export**: Player/recruiter CV export logic in `lib/generateCV.ts` and `lib/generateRecruiterCV.ts`.

## Integration Points
- **Cloudinary**: For uploads, configure via `.env` and use helpers in `lib/`.
- **Swiss Volley API**: Optional, see `.env` and `lib/` for usage.
- **External club sites**: See `lib/clubMatcher.ts` and `lib/clubsDatabase.ts` for linking logic.

## Examples & Patterns
- **Dynamic routing**: `app/players/[id]/page.tsx`, `app/hybrids/[id]/page.tsx`
- **Modals**: See `components/shared/BackgroundPickerModal.tsx`
- **Data import**: See `scripts/import-volleybox-data.ts` and `scripts/README.md`
- **Player video upload**: `components/player/VideoUpload.tsx`

## Tips for AI Agents
- Reference `README.md` and `scripts/README.md` for workflows and data flows.
- When adding new features, follow the Next.js app directory and TypeScript conventions.
- For new data models, update `prisma/schema.prisma` and run `npm run db:push`.
- Use Tailwind for all new UI.
- For Swiss-specific features, check `lib/` and `data/` for helpers and static data.

---
For questions, see `README.md` or open a GitHub issue.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/miocrobos)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/miocrobos)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
