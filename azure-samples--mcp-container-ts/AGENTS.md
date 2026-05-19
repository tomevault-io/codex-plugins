# Copilot Instructions for MCP Container TS

## Project Overview
- This project implements a Model Context Protocol (MCP) server using Node.js and TypeScript, designed to run locally or on Azure Container Apps.
- The MCP server acts as a bridge for AI models and tools, enabling communication and orchestration of tasks via HTTP endpoints.
- Key components: MCP HTTP server (`src/server.ts`), tool definitions (`src/tools.ts`), SQLite database integration (`src/db.ts`), and infrastructure as code (`infra/`).

## Architecture & Data Flow
- The server exposes endpoints for tool execution (add, list, complete, delete TODOs) and streams results using HTTP or SSE.
- Tools interact with an in-memory SQLite DB for state management.
- Configuration for local and remote servers is managed via `.vscode/mcp.json`.
- Deployment to Azure is automated using Bicep files and the Azure Developer CLI (`azd`).

## Developer Workflows
- **Local Development:**
  - Install dependencies: `npm install`
  - Start server: `npm run dev` (default port: 3000)
  - Database is ephemeral (in-memory SQLite).
- **Azure Deployment:**
  - Provision and deploy: `azd up`
  - Clean up: `azd down --purge --force`
- **Testing:**
  - Use MCP Inspector (`npm run inspect`) or VS Code MCP client integration via `.vscode/mcp.json`.
  - Authentication uses a demo token (`abc`).

## Project-Specific Conventions
- Tool outputs are always structured as `{ content: string[] }` for consistency.
- All tool logic is centralized in `src/tools.ts` and follows a simple input/output schema.
- Server configuration (local/remote, tokens) is managed in `.vscode/mcp.json`.
- Infrastructure is defined in Bicep (`infra/`), with parameters in `main.parameters.json`.
- No persistent database; all state is lost on server restart unless customized.

## Integration Points
- External communication via HTTP/SSE endpoints (`/mcp`, `/sse`).
- Azure Container Apps for cloud deployment; local Docker support via `Dockerfile` and `docker-compose.yml`.
- MCP Inspector and VS Code MCP client for tool invocation and inspection.

## Key Files & Directories
- `src/server.ts`: MCP HTTP server entry point
- `src/tools.ts`: Tool definitions and logic
- `src/db.ts`: SQLite DB integration
- `infra/`: Azure infrastructure (Bicep)
- `.vscode/mcp.json`: MCP server/client configuration
- `Dockerfile`, `docker-compose.yml`: Containerization
- `README.md`: Full project documentation and workflow details

## Example Patterns
- Tool definition:
  ```typescript
  {
    name: "http_add_todo",
    inputSchema: { ... },
    outputSchema: { content: string[] },
    async execute({ title }) {
      // ...
      return { content: ["Added TODO: ..."] };
    }
  }
  ```
- Server start:
  ```bash
  npm run dev
  # MCP endpoint: http://localhost:3000/mcp
  ```
- Azure deploy:
  ```bash
  azd up
  # Output: https://<env>.<container-id>.<region>.azurecontainerapps.io/mcp
  ```

---

If any section is unclear or missing, please provide feedback so this guide can be improved for future AI agents.

---
> Source: [Azure-Samples/mcp-container-ts](https://github.com/Azure-Samples/mcp-container-ts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-19 -->
