# Copilot Instructions for AIEngine--Gradia

## Project Overview
- **AIEngine--Gradia** (LumenAI/Perplexica) is a Next.js-based AI assistant platform combining chat, research, and widgets.
- The system supports dual modes: direct chat (LLM-only) and research (web, files, widgets, citations).
- Major features: conversational AI, web/academic/discussion search, file analysis, widgets (weather, stocks, etc.), persistent chat history, and a modern UI.

## Architecture & Data Flow
- **Frontend**: React components in `src/components/` and `src/app/`.
- **API**: Next.js API routes in `src/app/api/` (e.g., `/api/chat`, `/api/search`).
- **Agents**: Core logic in `src/lib/agents/`:
  - `chat/`: Direct LLM chat (no research)
  - `search/`: Classification, research, widgets, answer synthesis
- **Session & Storage**: Session management in `src/lib/session.ts`, chat/message persistence via Drizzle ORM (`src/lib/db/`).
- **Prompts**: Prompt templates in `src/lib/prompts/`.
- **Types**: Shared types in `src/lib/types.ts`.
- **Widgets**: Modular helpers in `src/lib/agents/search/widgets/`.

## Key Patterns & Conventions
- **Agent Pattern**: All user queries are routed through either `ChatAgent` or `SearchAgent` (see `src/lib/agents/`).
- **Classification**: Each query is classified to determine if research/widgets are needed (see `src/lib/agents/search/classifier.ts`).
- **Parallel Execution**: Research and widgets run in parallel before answer synthesis.
- **Citations**: Answers include citations, rendered in the UI.
- **Modes**: `speed`, `balanced`, `quality` modes control LLM/research tradeoff.
- **File Uploads**: User files are stored in `data/uploads/` and indexed for semantic search.

## Developer Workflows
- **Dev server**: `npm run dev` (Next.js)
- **Build**: `npm run build`
- **Start**: `npm run start`
- **Lint**: `npm run lint`
- **Format**: `npm run format:write`
- **Database**: Drizzle ORM migrations in `drizzle/`
- **Config**: See `drizzle.config.ts`, `next.config.mjs`, and `.env` (if present)

## Integration Points
- **LLMs**: OpenAI, Google, and others via `@ai-sdk/openai-compatible`, `@google/genai`, etc.
- **Search**: Meta search backend, SearxNG config in `searxng/`
- **Widgets**: Weather, stocks, etc. in `src/components/Widgets/` and `src/lib/agents/search/widgets/`

## Examples
- To add a new widget: implement in `src/lib/agents/search/widgets/`, register in `WidgetExecutor`.
- To add a new agent: follow the `ChatAgent`/`SearchAgent` pattern in `src/lib/agents/`.
- To extend prompts: edit/add in `src/lib/prompts/`.

## References
- [docs/architecture/README.md](../docs/architecture/README.md) — high-level architecture
- [docs/architecture/WORKING.md](../docs/architecture/WORKING.md) — data flow and answer lifecycle
- [src/lib/agents/](../src/lib/agents/) — agent logic
- [src/app/api/](../src/app/api/) — API endpoints
- [src/components/](../src/components/) — UI components

---
For more details, see the referenced docs and explore the `src/lib/agents/` directory for agent orchestration logic.

---
> Source: [krishna070502/LumenAI](https://github.com/krishna070502/LumenAI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-19 -->
