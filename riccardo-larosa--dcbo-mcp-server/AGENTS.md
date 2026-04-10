# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Multi-tenant MCP (Model Context Protocol) server that acts as an OAuth2 authorization server proxy for Docebo LMS. Enables MCP clients (Claude, ChatGPT, MCP Inspector) to discover and authenticate with multiple Docebo tenants through a single endpoint. Uses Express v5, MCP protocol version 2025-03-26, and requires Node >= 22.

## Commands

| Command | Purpose |
|---|---|
| `npm run dev` | Start dev server with hot reload (tsx watch) |
| `npm run build` | Compile TypeScript (`tsc`) to `dist/` |
| `npm start` | Run compiled server (`node dist/server.js`) |
| `npm run type-check` | Type-check without emitting |
| `npm test` | Run all tests once |
| `npm run test:watch` | Run tests in watch mode |
| `npm run test:coverage` | Run tests with V8 coverage |
| `npx vitest run src/docebo.test.ts` | Run a single test file |

No linter or formatter is configured.

## Architecture

All source lives in `src/`. Tests are co-located (`*.test.ts` next to source).

**`server.ts`** — Entry point. Express v5 app that registers all routes and middleware.

**`config.ts`** — Loads/validates env vars into `appConfig` (port, publicUrl, allowedOrigins, allowLocalDev). Evaluated eagerly at import time.

**`tenants.ts`** — Multi-tenant credential management. Reads `TENANT_<UPPERCASE>_CLIENT_ID/CLIENT_SECRET/REDIRECT_URI` from env. Converts tenant slug (hyphens) to env var format (underscores, uppercase). Exports `getTenantCredentials`, `getTenantConfig`, `getTenantOAuthEndpoints`, `getTenantApiUrl`, `listConfiguredTenants`.

**`oauth-proxy.ts`** — OAuth2 authorize/token proxy. Handles redirect to Docebo, callback, and token exchange. Encodes tenant + original state + client redirect_uri into the OAuth state parameter as base64url JSON. Resolves virtual client credentials to real tenant credentials.

**`virtual-clients.ts`** — POC Dynamic Client Registration. File-based storage (`virtual-clients.txt`, pipe-delimited). Generates deterministic secrets via HMAC-SHA256 using `DCR_SERVER_SECRET`.

**`mcp.ts`** — MCP JSON-RPC handler. Implements `initialize`, `tools/list`, `tools/call`. Three tools: `docebo_list_users`, `docebo_harmony_search`, `docebo_enroll_user`.

**`docebo.ts`** — Docebo API client. `listUsers` calls `/manage/v1/user`, `enrollUser` calls `/learn/v1/enrollments`, `harmonySearch` chains: bootstrap → geppetto auth → start session → message stream (SSE parsing).

### Request Flow

```
MCP: POST /mcp/:tenant (Bearer token) → validateOrigin → extractBearerToken → JSON-RPC dispatch → mcp.ts → docebo.ts → Docebo API

OAuth: GET /.well-known/oauth-authorization-server → discovery
       GET /mcp/:tenant/oauth2/authorize → redirect to Docebo
       GET /oauth/callback → redirect back to client with code
       POST /mcp/:tenant/oauth2/token → proxy to Docebo with real credentials
```

## Key Conventions

- **ESM modules**: All TypeScript imports use `.js` extension (e.g., `import { appConfig } from './config.js'`)
- **Logging**: Console-based with `[Module]` prefixes (`[Server]`, `[OAuth Proxy]`, `[MCP]`, `[Docebo]`)
- **Error responses**: JSON-RPC error codes (-32600 through -32603)
- **Stateless**: No sessions; state flows through OAuth parameters and Bearer tokens

## Testing Patterns

- Vitest with globals enabled (no need to import `describe`, `it`, `expect`)
- Mock modules with `vi.mock()`, functions with `vi.fn()`, HTTP with `global.fetch = vi.fn()`
- For `config.ts` tests: use `vi.resetModules()` and dynamic `import()` since config evaluates eagerly
- Tenant tests set `process.env.TENANT_*` vars in `beforeEach` and clean up in `afterEach`

## Environment Variables

Copy `.env.example` to `.env`. Key variables:

- `SERVER_PUBLIC_URL` — Required. Public URL of the server
- `PORT` — Server port (default: 3000)
- `ALLOWED_ORIGINS` — Comma-separated origins or `*` (default: `*`)
- `ALLOW_LOCAL_DEV` — Set `true` to disable origin checking
- `TENANT_<ID>_CLIENT_ID` / `CLIENT_SECRET` / `REDIRECT_URI` — Per-tenant OAuth2 credentials
- `DCR_SERVER_SECRET` — Secret for virtual client HMAC (default: `'default-secret-change-me'`)

Tenant ID format: URL slug `riccardo-lr-test` → env var prefix `TENANT_RICCARDO_LR_TEST_`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/riccardo-larosa)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/riccardo-larosa)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
