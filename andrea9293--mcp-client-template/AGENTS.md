# Copilot Coding Agent Instructions for MCP Client Template

## Project Overview
- This is a template for MCP (Model Context Protocol) client projects, combining a Vite + TypeScript frontend and an Express backend.
- The project supports OAuth2.1 authentication for secure server access.
- The `data/` folder is runtime-only and should not be modified or used for logic.

## Architecture & Data Flow
- **Frontend (`src/`, `index.html`)**: UI for adding MCP servers, sending commands, and displaying output. Communicates with backend via HTTP requests.
- **Backend (`server/`)**: Express REST API for managing MCP servers, forwarding commands, and handling authentication. Integrates with `@modelcontextprotocol/sdk` for MCP communication.
- **Integration**: Frontend and backend communicate over HTTP (default backend port: 3001).

## Developer Workflows
- **Install dependencies**: `npm install`
- **Start frontend**: `npm run dev`
- **Start backend**: `npm run dev:server`

## Key Patterns & Conventions
- **Server Management**: Servers are added via POST `/add-server` (see `server/index.ts`).
- **Frontend State**: UI state is managed in `src/main.ts` using direct DOM manipulation and fetch API.
- **Authentication**: OAuth2.1 flows are handled via backend logic and `public/oauth-callback.html`.
- **TypeScript Strictness**: Project uses strict TypeScript settings (`tsconfig.json`).
- **No direct logic in `data/`**: All runtime data is managed via backend APIs.

## External Dependencies
- Uses `@modelcontextprotocol/sdk` for MCP protocol communication.
- Relies on Express, CORS, and TypeScript for backend and build tooling.

## Examples
- To add a new MCP server, POST to `/add-server` with `{ id, command, type, args }`.
- To send a command, use the frontend form which calls backend endpoints.
- Extend backend logic in `server/index.ts` and related files for new API routes or authentication flows.

## File References
- Frontend: `src/main.ts`, `src/style.css`, `index.html`
- Backend: `server/index.ts`, `server/authTokensStore.ts`, `server/mcpTypes.ts`
- Static/OAuth: `public/oauth-callback.html`, `public/vite.svg`
- Config: `package.json`, `tsconfig.json`

---

If any conventions or workflows are unclear, please request clarification or examples from the maintainer.

---
> Source: [andrea9293/mcp-client-template](https://github.com/andrea9293/mcp-client-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-14 -->
