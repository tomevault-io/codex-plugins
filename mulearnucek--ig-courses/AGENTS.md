# Copilot Instructions

## Project Overview
IG is a Next.js 16 educational platform. Course content lives in a **separate GitHub repository** (configured via env vars), fetched at build/request time via the GitHub API. The frontend renders courses but never stores content locally.

## Architecture: Course Data Flow

```
GitHub Repo (content source) — one branch per course
  └── lib/github.ts  →  lib/courses.ts  →  app/classroom/[courseId]/[classId]/page.tsx (RSC)
                                                  └── ClassroomContent.tsx (client)
```

- `lib/github.ts` lists all repo branches (excluding `main`) via the GitHub Branches API — each branch is one course, and the branch name is the course slug/ID.
- Per branch, it fetches a full recursive tree and parses `index.json` files at 3 depths (root/section/module), assembles typed `Course` objects. GitHub API calls use `next: { revalidate: 3600 }`.
- `lib/courses.ts` wraps `github.ts` with an in-memory cache (`cachedCourses`) for SSG.
- Route params `courseId` / `classId` map directly to the branch name and module folder name respectively.

**Required env vars** (see `.env.example`): `GITHUB_REPO_OWNER`, `GITHUB_REPO_NAME`, `GITHUB_TOKEN`.

## Course Repository Structure (in the content GitHub repo)

Each course lives on its own branch (branch name = course slug). Content is at the **root** of the branch (no top-level course directory):

```
index.json                          # course metadata: title, description, authors, tags, difficulty, duration
thumbnail.jpeg                      # optional
[section-id]/index.json             # title, modules: ["module-id-1", "module-id-2"]  ← ordering
[section-id]/[module-id]/index.json # title, type, exerciseType?, validation?
[section-id]/[module-id]/video.mp4  # for "lecture" type
[section-id]/[module-id]/guide.md   # for "article" type (or supplementary)
[section-id]/[module-id]/actions.csv   # optional, lecture action timeline
[section-id]/[module-id]/resources.csv
```

Module `type` values: `"lecture"` | `"article"` | `"exercise"`. Folder names must be **kebab-case, no numeric prefixes**. The `modules` array in `section/index.json` defines display order.

See `docs/course-generation-prompt.md` for the full content-generation specification.

## Actions System (CSV-driven video interactivity)

`actions.csv` synchronizes UI actions to video timestamps (milliseconds). Timestamp `-1` = on mount.

```csv
-1,checkpoint,"Start","Welcome!"
16000,minimise
28000,highlight,chat_window,3000
34000,prompt,text,"Any questions?"
40000,prompt,checkbox,"Pick topics","React","Next.js"
```

Supported actions: `maximise`, `minimise`, `highlight`, `switch-theme`, `prompt`, `checkpoint`, `text`, `task`, `highlight-resource`.

- All action functions receive `ActionContext` (see `components/actions/types.ts`) as first arg.
- Each action is its own file under `components/actions/`; `Actions.tsx` orchestrates parsing + dispatch.
- `prompt` and `checkpoint` pause the video; playback resumes on user interaction.
- Full spec: `docs/actions-generation.md`.

## UI Stack & Conventions

- **Tailwind CSS v4** (PostCSS plugin, not v3 config object)
- **shadcn/ui** components in `components/ui/` (Radix UI based); add new ones via `components.json`
- **next-themes** for dark/light; wrapped in `ThemeProvider` at root layout
- **framer-motion** for animations in `ClassroomContent.tsx`
- **react-markdown** for rendering article/guide content
- **lucide-react** for icons

Path alias `@/` maps to repo root (e.g., `@/components/actions/types`).

## Key Routes

| Route | Type | Notes |
|---|---|---|
| `/classroom` | RSC | Course listing |
| `/classroom/[courseId]` | RSC | Course overview with `CourseSidebar` |
| `/classroom/[courseId]/[classId]` | RSC → client | Fetches `ClassItem`, renders `ClassroomContent` |

`ClassroomContent.tsx` is the main client component; it handles video state, action playback, markdown loading, and navigation between classes.

## Backend (Planned, Not Yet Implemented)
A separate Fastify/MongoDB/Better-Auth API server is spec'd in `docs/backend-specification.md` for auth, progress tracking, and rewards. The frontend currently has no auth layer.

## Dev Commands

```bash
bun dev      # start dev server (bun is the preferred package manager)
bun build
bun lint
```

---
> Source: [mulearnucek/ig-courses](https://github.com/mulearnucek/ig-courses) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-29 -->
