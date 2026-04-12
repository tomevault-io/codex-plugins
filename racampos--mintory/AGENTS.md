
# API Contracts Enforcement

**Backend REST + SSE:** Must match `docs/context/backend_api_spec.md`.

- `/runs` start, `/runs/{id}` status, `/runs/{id}/stream` SSE, `/runs/{id}/resume` decisions.

**MCP server:** Must match `docs/context/mcp_tools_spec.md`.

- All write calls return **PreparedTx**, never send transactions.

**Validation:** Add request/response validators and integration tests that assert exact shapes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/racampos)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/racampos)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
