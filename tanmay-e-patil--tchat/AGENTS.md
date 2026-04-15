# AI Assistant Project Instructions

Concise, project-specific guidance for AI coding agents working on this repo. Focus on THESE patterns; avoid generic boilerplate.

## 1. Tech Stack & Runtime
- Framework: Next.js 15 (App Router) with React 19, TypeScript.
- Styling: Tailwind CSS v4, utility merge via `cn` helper (`src/lib/utils.ts`).
- UI primitives: Radix UI + shadcn-inspired component set under `src/components/ui/**` and `src/components/ui/shadcn-io/**`.
- Auth: Clerk (`@clerk/nextjs`) integrated at root layout via `<ClerkProvider>`.
- Theming: `next-themes` wrapped by `ThemeProvider` (`src/components/theme-provider.tsx`).
- AI-related UI components (prompt input, message rendering) live in `src/components/ui/shadcn-io/ai/` but only `prompt-input.tsx` is currently wired in.

## 2. App Structure & Routing
- Root layout: `src/app/layout.tsx` sets global providers (Clerk, fonts, theme) and global CSS.
- Route groups:
  - `(chat)` group: main chat interface at `src/app/(chat)/page.tsx` using sidebar + prompt input.
  - `login` route: custom layout plus Clerk `<SignIn/>` (`src/app/login/*`).
- Keep new feature pages inside meaningful route groups for isolation (e.g. `(settings)`, `(admin)`); inherit providers automatically.

## 3. State & Interaction Patterns
- Predominantly client components (`"use client"`) for interactive UI (prompt input, sidebar, login page).
- Form submission for chat uses custom `onSubmit` handler; status cycles through: `submitted` -> `streaming` -> `ready`. Reuse this status enum for consistent UX.
- Keyboard behavior: Prompt textarea submits on Enter, newline with Shift+Enter (see `PromptInputTextarea`). Match this pattern for any additional multiline prompt inputs.
- Sidebar uses a context (`SidebarProvider`) with responsive behavior (mobile sheet vs desktop). Toggle shortcut: Cmd/Ctrl + `b`. When extending sidebar features, consume `useSidebar()` rather than duplicating responsive logic.

## 4. Styling & Class Composition
- Always compose Tailwind classes via `cn()` when conditional classes are needed to avoid duplicates.
- Variant-driven components use `class-variance-authority` (e.g. `button.tsx`, sidebar menu buttons) — extend variants by editing the `cva` call, not by layering ad‑hoc classes at call sites.
- Maintain data attributes already used for stateful styling (e.g. `data-state`, `data-slot`, `data-sidebar`, `data-active`). When adding new interactive elements in sidebar, follow existing naming for predictable theme/styling.

## 5. Sidebar Architecture Essentials
- File: `src/components/ui/sidebar.tsx` (large). Key exported building blocks: `SidebarProvider`, layout components (`Sidebar`, `SidebarInset`, `SidebarTrigger`), structural grouping (`SidebarGroup*`, `SidebarMenu*`).
- Mobile vs desktop handled internally; do NOT manually media-query in consumer components—just compose primitives.
- Collapsibility handled via data attributes; respect `data-collapsible` & `data-state` rather than adding new collapse logic.

## 6. Theming & Fonts
- Fonts loaded through `next/font/google` (Geist). If adding fonts, mirror pattern in root layout and extend CSS variables (`--font-*`).
- Theme switching handled by `ModeToggle` (in `src/components/ui/mode-toggle.tsx` if/when implemented). Use `className` theme tokens (e.g. `bg-background`, `text-muted-foreground`) instead of hard-coded colors.

## 7. Auth Integration (Clerk)
- Auth context is globally available; use `<SignedIn>` / `<SignedOut>` wrappers as in chat page header.
- For protected routes, wrap UI fragments rather than building a custom auth gate unless role logic becomes necessary.

## 8. Adding AI / Chat Logic
- Current chat is UI-only (simulated timeouts). To inject real AI:
  - Create server action or API route (e.g. `src/app/api/chat/route.ts`) using chosen provider SDK.
  - Stream updates: integrate with `ai` package stream helpers; wire to status transitions already present.
  - Keep model selection controlled by `model` state; pass selected model to backend.
- Reuse `PromptInput*` subcomponents for consistency; avoid duplicating prompt form markup.

## 9. Conventions & Naming
- Components: PascalCase file per component; colocate related minor exports in same file (e.g. prompt input variants in one file).
- Hooks under `src/hooks/` start with `use*`; keep them framework-agnostic and client-only if they access `window`.
- Utility helpers live in `src/lib/`.
- Prefer explicit export lists over default except for main React pages/layouts.

## 10. Build & Dev Workflows (Bun-first)
- Use Bun for all scripts (fast install & runtime). Examples:
  - Install deps (fresh clone): `bun install`
  - Dev (Turbopack): `bun dev` (alias for `bun run dev`)
  - Build: `bun run build`
  - Start production build: `bun start`
  - Lint: `bun run lint`
- Other package managers (npm/pnpm/yarn) should be avoided in commits to keep a single lockfile (`bun.lock`).
- If adding a new script, prefer `bun run <script>` pattern in docs.
- Tests: none yet. When introducing, add a `test` script (e.g. Vitest: `"test": "vitest"`) and run with `bun test`.

## 11. When Extending
- Before adding a new UI primitive, check for an existing pattern in `src/components/ui/` or `shadcn-io/ai/` to avoid duplication.
- Keep accessibility: maintain `aria-*` and `sr-only` spans as seen in `SidebarTrigger` and menu buttons.
- For responsive behavior, prefer existing hooks/contexts; only create new contexts for cross-cutting state.

## 12. Quick Examples
- Conditional classes: `className={cn("p-2", isActive && "bg-accent")}`.
- New sidebar section: wrap items with `SidebarGroup` -> `SidebarGroupLabel` + `SidebarMenu` / `SidebarMenuItem` + `SidebarMenuButton`.
- Prompt submit handler template mirrors existing `handleSubmit` in chat page.

## 13. Non-Goals / Avoid
- Do not introduce global state libraries (Redux, Zustand) unless complexity warrants; current context/hooks suffice.
- Avoid hard-coded pixel colors; rely on theme tokens.
- Don't bypass variant systems with inline style overrides unless absolutely necessary.

Feedback welcome: Clarify real AI backend integration, testing approach, or sidebar extension patterns if needed.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tanmay-e-patil) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
