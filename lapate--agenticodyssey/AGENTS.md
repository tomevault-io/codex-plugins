# GitHub Copilot Instructions — Agentic Odyssey

This file provides persistent context for GitHub Copilot when working in this repository.

---

## Project Purpose

**Agentic Odyssey** is an instructor-led, hands-on student workshop teaching participants how to build, orchestrate, and evaluate AI agents using **Microsoft Azure AI Foundry**. Students run the lab in **GitHub Codespaces** (browser-based VS Code). The audience is technical but may be new to Azure and AI agents.

All documentation, scripts, and code should be written with **clarity and simplicity** as the primary goal — students need to succeed on their first try.

---

## Azure Conventions

| Setting | Value |
|---------|-------|
| **Resource Group** | `agenticodyssey-rg` |
| **Location / Region** | `westus3` |
| **MCP Server Image** | `ghcr.io/lapate/agenticodyssey/mcp-server` |
| **MCP Server Port** | `8000` |
| **MCP Transport** | SSE — endpoint path is `/sse` |

- All Azure resources in this workshop deploy to **`agenticodyssey-rg`** in **`westus3`**.
- Do not use other regions or resource group names unless the user explicitly asks.
- Resource names and resource groups are **hard-coded** in scripts (not overridable via env vars) to reduce student confusion.

---

## Repository Structure

```
AgenticOdyssey/
├── .github/
│   ├── copilot-instructions.md   ← This file
│   └── workflows/
│       └── build-mcp-server.yml  ← Builds & pushes MCP server Docker image
├── data/                         ← JSON datasets + News Story markdown files
├── labs/                         ← Student-facing lab guides (Markdown)
├── mcp_server/                   ← FastMCP server source + Dockerfile
│   ├── server.py
│   ├── requirements.txt
│   ├── Dockerfile
│   ├── .dockerignore
│   ├── chicken_daily_orders_financials.json
│   └── chicken_hourly_store_sales.json
├── scripts/                      ← Deployment & setup bash scripts
│   ├── setup.sh                  ← Auto-runs in Codespaces (devcontainer postCreate)
│   ├── deploy-mcp-server.sh      ← Deploys MCP server to Azure Container Instance
│   └── create-azure-ai-search.sh ← Deploys Azure AI Search + uploads documents
├── old/                          ← Previous lab materials (kept for reference)
├── new/                          ← New lab notebooks in development
└── README.md                     ← Participant guide (main student-facing doc)
```

---

## MCP Server Details

- Built with **FastMCP**, SSE transport, bound to `0.0.0.0:8000`
- Exposes **10 CRUD tools** over two datasets:
  - `list/get/create/update/delete_daily_financial` — daily orders & financials
  - `list/get/create/update/delete_hourly_sale` — hourly store sales
- JSON files in `mcp_server/` are the data store (no database)
- Every record has a `uuid` `id` field for CRUD operations
- GitHub Action rebuilds the Docker image on every push to `mcp_server/**`

---

## Script Conventions

- All scripts are **bash** (`.sh`)
- Line endings must be **LF** (Unix) — not CRLF. This matters because scripts run inside Linux containers/Codespaces.
- Student-editable variables go at the **top of the script** in a clearly marked `── STUDENT: Edit these variables ──` section
- Fixed/shared values (resource group, location) are **hard-coded constants**, not `${VAR:-default}` patterns
- Scripts must include **prerequisite checks** (az CLI installed, az login active) before doing any work
- Scripts print a clear **summary block** at the end with all relevant output values (IPs, endpoints, keys)

---

## Lab / Documentation Conventions

- Lab guides live in `labs/` as Markdown files
- Lab file naming: `create-<thing>.md` or `build-<thing>.md`
- Use `📸 **HUMAN — DO THIS:**` as the screenshot indicator wherever a human needs to take a screenshot. This is intentional — do not change this format.
- Lab guides include:
  1. A brief "What You'll Need" section listing prerequisites with a table
  2. Numbered steps with explicit UI navigation instructions
  3. Screenshot markers at every key UI moment
  4. Test prompts / verification steps
  5. A Troubleshooting table at the end
- The `README.md` is the **participant guide** — it covers environment setup and links to individual labs
- Agent names follow PascalCase: `ZavaGroceriesInventoryAgent`, `SalesInsightsAgent`

---

## Branch & Merge Policy

- **Never merge a branch into `main` without explicit approval from the user.**
- When resolving merge conflicts between branches, always confirm the resolution strategy before committing.
- When updating a branch to be PR-ready, merge `main` into the feature branch (not the other way) so `main` stays clean.

---

## Workshop Flow (Labs in Order)

1. **Setup** (README.md) — Codespaces, `az login`, deploy MCP server, deploy AI Search
2. **Lab 1** (`labs/create-inventory-agent.md`) — Build `ZavaGroceriesInventoryAgent` in Foundry portal, connect MCP server
3. **Lab 2** (`labs/create-insights-agent.md`) — Build `SalesInsightsAgent` in Foundry portal, connect Azure AI Search + MCP server, produces structured markdown recommendations for human review or agent handoff
4. More labs TBD — Python orchestration wiring the two agents together, agent evaluation

---
> Source: [lapate/AgenticOdyssey](https://github.com/lapate/AgenticOdyssey) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-28 -->
