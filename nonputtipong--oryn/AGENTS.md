> This file is mirrored across CLAUDE.md, AGENTS.md, GEMINI.md, .cursorrules, .windsurfrules, and .clinerules so the same instructions load in any AI environment.

# Agent Instructions

You operate within a 3-layer architecture that separates concerns to maximize reliability. LLMs are probabilistic, whereas most business logic is deterministic and requires consistency. This system fixes that mismatch.

## The 3-Layer Architecture

**Layer 1: Directive (What to do)**  
- Basically just SOPs written in Markdown, live in `rules/`  
- Define the goals, inputs, tools/scripts to use, outputs, and edge cases  
- Natural language instructions, like you'd give a mid-level employee

**Layer 2: Orchestration (Decision making)**  
- This is you. Your job: intelligent routing.  
- Read directives, call execution tools in the right order, handle errors, ask for clarification, update directives with learnings  
- You're the glue between intent and execution. E.g you don't try scraping websites yourself—you read `rules/scrape_website.md` and come up with inputs/outputs and then run `execution/scrape_single_site.py`

**Layer 3: Execution (Doing the work)**  
- Deterministic Python scripts in `execution/`  
- Environment variables, api tokens, etc are stored in `.env`  
- Handle API calls, data processing, file operations, database interactions  
- Reliable, testable, fast. Use scripts instead of manual work. Commented well.

**Why this works:** if you do everything yourself, errors compound. 90% accuracy per step = 59% success over 5 steps. The solution is push complexity into deterministic code. That way you just focus on decision-making.

## Mandatory Agent Skills & Superpowers

To maximize performance, you have access to specialized skills. You MUST apply these automatically:

**1. Ultimate Superpower (`using-superpowers`)**
- **Trigger:** MUST be evaluated and activated when starting **ANY** conversation.
- **Path:** `.agent/skills/using-superpowers/SKILL.md`
- **Power:** Enforces the golden rule: If there is even a 1% chance a skill applies to a task, you MUST invoke it via the Skill tool BEFORE making any response or query. Do not rationalize skipping it. Process skills (how to think) take priority over implementation skills (how to execute).

**2. Lightning-fast Python Workflows (`uv-package-manager`)**
- **Trigger:** ALWAYS use this when working with Python dependencies or environments, as the user has `uv` installed globally. 
- **Path:** `.agent/skills/uv-package-manager/SKILL.md`
- **Power:** Grants 10-100x faster package installation and resolving than pip. Use it to execute deterministic backend scripts using `uv run` (bypassing the need to manually activate venvs), setup projects with `uv init`, manage dependencies with `uv add`/`uv sync`, and reproduce clean environments with `uv lock`. Never run pure pip or poetry; always optimize with `uv`.

## Operating Principles

**1. Check for tools first**  
Before writing a script, check `execution/` per your rules/directives. Only create new scripts if none exist.

**2. Always Check Rules Directory**
Before starting any specific task, you MUST read the relevant rule files in the `rules/` directory:
- **Core Workflows**: For the strict Pipeline (Plan -> Audit -> Implement -> Review), testing standards, and git flows, read `rules/workflow.md` and `rules/git-workflow.md`.
- **Domain Specific Instructions**: We have extensive codebase documentation. If your task involves it, you MUST read the corresponding rule file (e.g. `rules/forms.md`, `rules/ui-components.md`, `rules/admin.md`, `rules/security.md`, `rules/database.md`, `rules/testing.md`).
- **Research & Deep Dives**: Check `/docs/research/` for agent architecture research and best practices for configuring new workflows.

**3. Self-anneal when things break**  
- Read error message and stack trace  
- Fix the script and test it again (unless it uses paid tokens/credits/etc—in which case you check w user first)
- Update the rules with what you learned (API limits, timing, edge cases)  
- Example: you hit an API rate limit → you then look into API → find a batch endpoint that would fix → rewrite script to accommodate → test → update rules.

**4. Update rules as you learn**  
Rules are living documents. When you discover API constraints, better approaches, common errors, or timing expectations—update the rules. But don't create or overwrite rules without asking unless explicitly told to. Rules are your instruction set and must be preserved (and improved upon over time, not extemporaneously used and then discarded).

## Self-annealing loop

Errors are learning opportunities. When something breaks:  
1. Fix it  
2. Update the tool  
3. Test tool, make sure it works  
4. Update rules to include new flow  
5. System is now stronger

---

## Web App Development

### Tech Stack
When asked to create a web app, use:

- **Frontend**: Next.js (latest) + React + Tailwind CSS
- **Backend**: Supabase + Next.js API routes
- **Database**: Supabase (PostgreSQL)
- **Authentication**: Clerk
- **Payment**: Stripe
- **Hosting**: Vercel
- **CDN**: Cloudflare
- **Analytics**: PostHog
- **Monitoring**: Sentry
- **CI/CD**: GitHub Actions


### Brand Guidelines
- Before development, check for `brand-guidelines.md` in the project root
- If present, use the specified fonts and colors to maintain brand consistency

### Directory Structure for Applications

```text
project-root/
├── frontend/ # Next.js app
│   ├── app/ # Next.js App Router
│   │   └── api/ # Next.js API routes
│   ├── components/ # React components
│   ├── lib/ # Next.js utility functions and clients (Supabase, Stripe, etc.)
│   ├── public/ # Static assets
│   └── package.json
├── supabase/ # Local Supabase configuration and migrations (if needed)
├── .github/
│   └── workflows/ # GitHub Actions for CI/CD
├── rules/ # Markdown SOPs and rules
├── execution/ # Utility Python scripts
├── .tmp/ # Intermediate files
└── brand-guidelines.md # (optional) Fonts and colors
```

---

## File Organization

**Deliverables vs Intermediates:**  
- **Deliverables**: Google Sheets, Google Slides, or other cloud-based outputs that the user can access  
- **Intermediates**: Temporary files needed during processing

**Directory structure:**  
- `.tmp/` - All intermediate files (dossiers, scraped data, temp exports). Never commit, always regenerated.  
- `execution/` - Python scripts (the deterministic tools)  
- `rules/` - SOPs in Markdown and project rules/guidelines
- `.env` - Environment variables and API keys  
- `credentials.json`, `token.json` - Google OAuth credentials (required files, in `.gitignore`)

**Key principle:** Local files are only for processing. Deliverables live in cloud services (Google Sheets, Slides, etc.) where the user can access them. Everything in `.tmp/` can be deleted and regenerated.

## Summary

You sit between human intent (directives) and deterministic execution (Python scripts). Read instructions, make decisions, call tools, handle errors, continuously improve the system.

Be pragmatic. Be reliable. Self-anneal.
## Oryn Project-Specific Conventions

### 🤖 Oryn Tech Stack
- **Runtime**: Node.js + TypeScript (via `tsx`)
- **LLM**: OpenRouter API (default: Gemini 2.0 Flash)
- **Telegram**: grammY framework
- **Database**: SQLite (`better-sqlite3`) + Full-text search (FTS5) + Vector search (`sqlite-vec`)
- **Voice**: Groq Whisper API (STT), ElevenLabs API (TTS)
- **Google APIs**: Native fetch + OAuth2 (no googleapis SDK)
- **Real-time**: WebSocket (`ws`)
- **Scheduling**: `node-cron`
- **Encryption**: Node.js crypto (AES-256-GCM)

### 🏗️ Architecture & Core Systems
- **ReAct Loop**: Located in `src/agent.ts`. Receives messages, hybrid-retrieves context, calls from 39 tools, and asynchronously extracts facts.
- **Memory System**: Mem0-inspired three-tier architecture (`src/memory/`).
  1. *Working*: In-memory for the current conversation.
  2. *Episodic*: Offline conversation summaries stored in SQLite.
  3. *Semantic*: Vector + Keyword full-text + Knowledge Graph fact extraction.
- **Channels & APIs**: Unified Google Workspace integrations (Gmail, Calendar, Drive) located in `src/channels/`.
- **Canvas**: Live real-time dashboard at `localhost:3001` via WebSocket (`src/canvas/`).
- **MCP Bridge**: Dynamic connection to external Model Context Protocol servers (`src/mcp/`).
- **Security & Sandboxing**: Encrypted AES-256-GCM vault and optional Docker container shells (`src/security/`).

### 📁 Structure
- `src/` - Core application logic.
- `data/` - Auto-created SQLite databases.
- `notes/` - Markdown file storage.
- `scripts/` - Utilities (like `gmail_auth.ts`).

### ⌨️ Development Commands
- Start dev server: `npm run dev`
- Run Gmail OAuth: `npm run gmail:auth`

*(Note: Although global Web App Development rules exist above, disregard the Next.js/Supabase frontend constraints when working strictly within the Oryn Telegram bot environment.)*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nonputtipong)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/nonputtipong)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
