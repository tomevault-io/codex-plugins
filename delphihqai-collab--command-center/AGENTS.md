# Mission Control — AI Assistant Instructions

## Core Directive: Your Environment & Identity

You are running directly on the local machine of **Hermes**, the Chief of this team and the Director of the commercial agent fleet. 

You are not just a passive coding assistant; you are operating within the command center. You have the ability to execute `openclaw` commands to manage the system. **Most importantly, you can communicate directly with Hermes at any time by using the `openclaw tui` command.** If you need clarification, want to report a critical success, or need to escalate an issue, use `openclaw tui` to talk to Hermes directly.

## Project Identity

Orchestration dashboard for **Hermes** — a commercial AI agent fleet running on **OpenClaw**. Next.js 16 + Supabase. Dark theme. Port 9069 on a private Linux machine.

**Owner:** Delphi. Single-user system. No multi-tenancy.

## Stack

Next.js 16 App Router · React 19 · TypeScript (strict) · Tailwind CSS v4 · shadcn/ui · Supabase SSR · Recharts · date-fns · Lucide icons · Sonner · @dnd-kit

## OpenClaw — The Agent Runtime

OpenClaw is the self-hosted AI agent gateway on this machine. Gateway WebSocket on `ws://127.0.0.1:18789`.

- **Workspace:** Each agent gets an isolated directory with structured template files
- **Channels:** Discord as primary channel. Agents bound via `openclaw agents bind`
- **Sessions:** `main` (default) or `isolated` (parallel). Sessions route through the gateway
- **Subagents:** Parent agents spawn background sub-agent runs via `subagent` tool
- **Cron:** Jobs in `~/.openclaw/cron/jobs.json`. Types: `at`, `every`, `cron`. Targets: `main` or `isolated`

### Agent Fleet — Hermes Commercial

8 agents. Hermes = director. 7 sub-agents = workers + specialists.

| Slug | Type | Model | Workspace |
|------|------|-------|-----------|
| hermes | director | claude-sonnet-4-6 | `~/.openclaw/workspace/` |
| sdr | worker | claude-sonnet-4-6 | `~/.openclaw/workspace/teams/commercial/sdr/` |
| account-executive | worker | claude-sonnet-4-6 | `~/.openclaw/workspace/teams/commercial/account-executive/` |
| account-manager | worker | claude-sonnet-4-6 | `~/.openclaw/workspace/teams/commercial/account-manager/` |
| finance | specialist | claude-haiku-4-5-20251001 | `~/.openclaw/workspace/teams/commercial/finance/` |
| legal | specialist | claude-sonnet-4-6 | `~/.openclaw/workspace/teams/commercial/legal/` |
| market-intelligence | specialist | claude-haiku-4-5-20251001 | `~/.openclaw/workspace/teams/commercial/market-intelligence/` |
| knowledge-curator | specialist | claude-sonnet-4-6 | `~/.openclaw/workspace/teams/commercial/knowledge-curator/` |

### Workspace Template Files

Each agent workspace contains: `SOUL.md` (personality/mission), `IDENTITY.md` (role/model/capabilities), `AGENTS.md` (sub-agent roster), `TOOLS.md` (available tools), `USER.md` (owner context), `BOOTSTRAP.md` (first-run), `HEARTBEAT.md` (periodic check-in), `BOOT.md` (session startup), `memory/` (agent directory).

### OpenClaw CLI & Direct Communication

You have full access to manage the fleet and communicate with the Chief using the CLI:

`openclaw tui`                                 # DIRECT COMM LINE: Talk directly with Hermes (Chief)
`openclaw agents list|add|delete|bind|unbind|set-identity`
`openclaw cron list|add|edit|remove|run|runs`
`openclaw config|status`

### Dual Data Sources

| Page | Source | Notes |
|------|--------|-------|
| Dashboard, Agents, Pipeline | Supabase | Standard CRUD |
| Sessions | Gateway API (`localhost:18789`) | Falls back to Supabase |
| Memory | Filesystem (`/api/memory`) | Reads `~/.openclaw/workspace/` |
| Gateway | Gateway API (`localhost:18789/config`) | Config viewer |
| Cron | OpenClaw (`~/.openclaw/cron/jobs.json`) | `openclaw cron` CLI |
| Logs | journalctl + Supabase | System logs |

## Coding Style

- TypeScript strict — no `any`, no unnecessary `!` assertions
- Functional React components only — no class components
- Server Components by default — add `"use client"` only when browser APIs or event handlers are required
- Async Server Components for data fetching — no `useEffect` for data, no client-side fetching unless realtime
- `Promise.all()` for parallel Supabase queries on a single page
- Always destructure `.data` and `.error` from Supabase responses
- Use `?? []` or `?? null` fallbacks — never assume Supabase returned data

## Import Conventions

`import { createClient } from "@/lib/supabase/server";` // Server Supabase client
`import { createClient } from "@/lib/supabase/client";` // Browser Supabase client
`import type { Database } from "@/lib/database.types";` // Generated Database type
`import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card";` // UI components
`import { Bot, Target, Users } from "lucide-react";` // Icons

## File Structure

- Pages: `src/app/(app)/<route>/page.tsx`
- Server actions: `src/app/(app)/<route>/actions.ts`
- Page-specific components: `src/app/(app)/<route>/_components/`
- Shared components: `src/components/`
- shadcn/ui primitives: `src/components/ui/` — do not edit
- Supabase clients: `src/lib/supabase/client.ts` and `server.ts`
- Generated types: `src/lib/database.types.ts` — do not edit manually
- Agent memory paths: `src/lib/memory-paths.ts` — maps agent slugs to filesystem dirs

## Tailwind Conventions

Dark theme only. Zinc palette:
- Background: `bg-zinc-950`
- Cards/surfaces: `bg-zinc-900`
- Borders: `border-zinc-800`
- Primary text: `text-zinc-50`
- Secondary text: `text-zinc-400`
- Accent: `bg-indigo-600` / `text-indigo-400`

Status colours:
- Healthy/active/paid/approved: `emerald-500`
- Warning/idle/pending/in-progress: `amber-500`
- Critical/failed/rejected/urgent: `red-500`
- Offline/archived/disabled: `zinc-500`

## Naming Conventions

- Components: PascalCase (`StatusBadge`, `AgentCard`)
- Files: kebab-case (`status-badge.tsx`, `agent-card.tsx`)
- Database columns: snake_case (matches Supabase schema)
- TypeScript interfaces: PascalCase (`Agent`, `PipelineLead`, `Webhook`)
- Server actions: camelCase verb (`createLead`, `moveLead`, `toggleWebhook`)

## OpenClaw Integration Rules

- Route Handlers that call OpenClaw CLI: `execFile("openclaw", [...args])` — never `exec()`
- Route Handlers that read OpenClaw files: validate paths, prevent directory traversal
- Gateway API calls: `fetch("http://127.0.0.1:18789/...")` — local only, via API route
- Never call gateway from client-side code — always through Next.js API routes
- Never hardcode workspace paths in components — use `MEMORY_PATHS` from `src/lib/memory-paths.ts`

## Do Not

- Do not import server client in Client Components
- Do not import browser client in Server Components
- Do not modify `src/lib/database.types.ts` manually
- Do not modify files in `src/components/ui/`
- Do not use `getServerSideProps` or `getStaticProps` — this is App Router
- Do not expose `SUPABASE_SERVICE_ROLE_KEY` to the client bundle
- Do not write raw SQL in application code — use the Supabase query builder
- Do not add `console.log` in production paths — use structured error handling
- Do not skip the `.error` check from Supabase responses
- Do not use `exec()` for shell commands — always `execFile()`
- Do not expose the OpenClaw gateway port to external networks
- Do not hardcode agent workspace paths — use `MEMORY_PATHS`

## Migrations — Critical Notes

Connection string has special characters that break standard URL parsers. Never use `supabase db push` directly.
Use the Node pg script documented in CLAUDE.md under "Migrations — How to Apply".

After any schema change:
1. Create migration file in `supabase/migrations/<timestamp>_description.sql`
2. Apply via the Node pg script
3. Run `npx supabase gen types typescript --linked > src/lib/database.types.ts`
4. Rebuild the app (NEXT_PUBLIC_* vars must be present at build time)

## Build Process

Always build with env vars:
`NEXT_PUBLIC_SUPABASE_URL="..." NEXT_PUBLIC_SUPABASE_ANON_KEY="..." npm run build`
Then restart the systemd service: `systemctl --user restart command-center`

## Running

`npm run dev`                                    # Dev server on port 9069
`systemctl --user status command-center`         # Production service
`systemctl --user restart command-center`        # Restart after deploy
`journalctl --user -u command-center -f`         # Tail logs
`systemctl --user status openclaw`               # OpenClaw gateway
`openclaw agents list`                           # List agents
`openclaw cron list`                             # List cron jobs
`openclaw status`                                # Gateway health

---

## HERMES Guide — Current-State Document

`docs/HERMES.md` is a current-state reference that explains Mission Control to Hermes. Every section describes how things work **right now**. There is no changelog section.

After every implementation session, **update the relevant sections** of `docs/HERMES.md` to reflect any changes — do not append entries to the bottom. If a page's data source changed, update the Data Sources table and the page description. If a new component was added, update the File Structure section. Keep every section accurate.

---

## Session Completion Protocol — MANDATORY

Before ending ANY implementation session, you MUST complete this checklist in order. **If you are unsure about any of these steps, execute `openclaw tui` to consult with Hermes before proceeding.**

1. **Build passes** — run `next build` and confirm zero errors
2. **HERMES.md updated** — update the relevant sections in `docs/HERMES.md` to reflect changes
3. **Git commit** — `git add -A && git commit -m "<type>: <summary>"`
4. **Git push** — `git push`
5. **Service restart** (if code changed) — `systemctl --user restart command-center`

⚠️ If you skip any of these steps, the session is considered incomplete.

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/delphihqai-collab)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/delphihqai-collab)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
