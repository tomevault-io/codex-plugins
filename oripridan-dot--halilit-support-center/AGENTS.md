# Repository Instructions & Context (v9.7.6 — Level 6)

## ⚠️ THE FUNDAMENTAL LAW — Three Source Rules (backend/source_rules.py)

**These rules are the FOUNDATION of the entire application. Without them, the app has NO VALUE.**

### The Three Authorized Data Sources

| #   | Source         | Owner   | Owns                                                                      |
| --- | -------------- | ------- | ------------------------------------------------------------------------- |
| 1   | **Commercial** | Halilit | Golden List, prices (IL+Eilat), SKUs, product existence                   |
| 2   | **Official**   | Brand   | Titles, descriptions, specs, media, documentation (official product page) |
| 3   | **Contextual** | Reviews | Pros/cons, real-world experience, ratings (3+ trusted review sites)       |

### Zero Tolerance Policy

- **NO synthesized/generated data** — empty fields are BETTER than fake fields
- **NO mock data** in any pipeline stage
- **NO AI-generated specs** presented as real specs
- **NO AI-generated reviews** presented as real reviews
- **NO fallback to simulated data** — if a source fails, the product stays incomplete
- Each source has **strict field ownership** — only the owner can set its fields
- **Cross-validation**: confidence benefits from multiple sources agreeing

**See `backend/source_rules.py` for enforcement.**

---

## Project Overview

**Halilit Support Center v9.7.6 (Chief)** — JIT product intelligence platform for musical instruments. Operator Console UI for inventory management and product support.

- **Architecture**: Spec-driven Dark Factory. Specs in `specs/` are the **input**; code is the **output**. Never write code without reading the relevant spec first.
- **Workflow**: `specs/interface/` → AI implements → verify outcome in app. See `OPERATOR_CONSOLE_SPEC.md` and `docs/WORKFLOW.md`.
- **Frontend**: React 18 + Vite + TypeScript + Zustand + React Query + Tailwind CSS. Three views: Dashboard, Inventory, ProductDetail.
- **Backend**: Python 3.11+ + FastAPI + google-genai (Gemini 2.0 Flash). Conductor CLI for data pipeline.
- **Repo strategy**: Lean — generated data (brand JSONs, graph snapshot) is gitignored. Only source code and static assets are tracked.

---

## Running the System

```bash
# Preferred — Master Factory Controller (requires GEMINI_API_KEY):
export GEMINI_API_KEY="your-key"        # or GOOGLE_API_KEY
python factory.py init                  # One-time: create folder structure
python factory.py status                # Check environment health
python factory.py design "description" [category]  # Architect: generate a spec
python factory.py build <spec_path>     # Builder: materialise spec → code
python factory.py start                 # Launch backend (8000) + frontend (5173)

# Legacy / direct:
./factory_reset.sh               # Clean start (backend + frontend)
./factory_reset.sh --rebuild     # Force rebuild catalog cache
PYTHONPATH=. python backend/conductor_main.py dev   # Both servers
PYTHONPATH=. python backend/server.py               # Backend only (port 8000)
cd frontend && pnpm dev                             # Frontend only (port 5173)
```

**Note:** `factory.py build` takes a spec file path (e.g. `specs/interface/dashboard.md`). `python factory.py design` writes specs to `specs/interface/` by default. `GEMINI_API_KEY` (or `GOOGLE_API_KEY`) is required for `design` and `build`.

---

## Tech Stack

- **Backend**: Python 3.11+, FastAPI, google-genai SDK, Pydantic v2, Pillow
- **Frontend**: React 18, TypeScript 5, Vite 5, Zustand 5, React Query 5, Tailwind CSS 3.4, Framer Motion
- **Catalog**: Built by `product_normalizer.build_catalog()` from `frontend/public/data/*.json`
- **Graph**: ProductGraph (families, relationships); persisted to `backend/data/graph/product_graph.json`

---

## File Structure (v9.7.6)

```
backend/
├── source_rules.py           # ⚠️ THE LAW — Three Source Rules (read first!)
├── server.py                 # FastAPI: catalog, JIT, MCP
├── conductor_main.py         # CLI: skeleton-sync, commercial-ingest, enrich, sync, dev, server
├── product_normalizer.py     # build_catalog(), graph pipeline
├── product_graph.py          # ProductGraph, families, relationships
├── product_graph_store.py    # JSON snapshot
├── jit_agent.py              # On-demand product intelligence (SSE streaming)
├── unified_data_service.py   # Sync engine
├── factory/                  # Builder Agent (spec → code), agent_core.py
├── factory_supervisor.py     # Foreman: orchestrates Builder Agent runs
├── hierarchy/                # Product hierarchy: models, service, api, validation
├── api/                      # mcp_router (FastAPI)
├── ingestion/                # halilit_page_scraper, relationship_*, taxonomy, data_models
├── mcp/                      # MCP servers (catalog_db, ui_bridge, …)
├── scripts/                  # enrich_catalog, generate_search_index, …
├── config/                   # init_db.sql, mcp_servers.json
└── data/                     # graph/, ingestion/, jit_cache/ (gitignored)

frontend/src/
├── App.tsx                   # Shell: sidebar + 3-view router (Dashboard, Inventory, ProductDetail)
├── components/
│   ├── views/                # DashboardView, InventoryView, ProductDetailView, IngestionStatusView
│   ├── cockpit/              # ProductRelations, VerdictCard, TrustedConsensus, FieldNotes, ExplorationDock
│   ├── GlobalSearch.tsx      # Search bar (header)
│   ├── ImageWithFallback.tsx # Image component with fallback
│   └── ui/                   # GlobalErrorBoundary (ui primitives)
├── hooks/
│   ├── useConductorCatalog.ts  # Catalog data from /api/conductor/catalog
│   └── useJITIntelligence.ts   # JIT streaming intelligence
├── store/
│   └── navigationStore.ts    # App navigation state (Zustand)
└── types/
    ├── index.ts              # Canonical frontend types
    └── generated.ts          # Backend-generated types

specs/
├── interface/                # ← CANONICAL UI SPECS
│   ├── 01_operator_dashboard.md
│   ├── 02_inventory_grid.md
│   └── 03_product_intelligence.md
├── data_pipeline/            # Ingestion rules, relationship logic
├── behavior/                 # Search scenarios (Playwright)
├── 01_data/                  # Compliance, halilit_api, official_scout, catalog_organizer
└── pricing_logic.md

OPERATOR_CONSOLE_SPEC.md      # Master spec (Level 6 workflow)
docs/                         # Dev documentation (QUICK_START, WORKFLOW, ARCHITECTURE, etc.)
```

---

## The Dark Factory Workflow

1. **Spec first**: Update or create the relevant spec in `specs/interface/` or `specs/data_pipeline/`.
2. **Prompt AI**: "Read `specs/interface/02_inventory_grid.md`. Rewrite `InventoryView.tsx` to satisfy the spec."
3. **Verify**: Check the outcome in the running app against the Behavior Scenarios in the spec.
4. **Fix spec, not code**: If outcome fails, amend the spec and re-prompt.

**Never write code without a spec. Never fix code by hand when the spec can be clarified.**

---

## Code Standards

### Frontend (React/TypeScript)

- **Types**: Import from `types/index.ts` (canonical; generated from backend when applicable).
- **State**: Zustand for app state, React Query for server state (catalog).
- **Styling**: Tailwind CSS; dark theme (e.g. slate-900, blue-500).
- **Components**: Functional components with hooks only (class only for ErrorBoundary).
- **Data**: All product data from `/api/conductor/catalog` (useConductorCatalog).
- **NEVER** leave a file empty or &lt; 100 bytes.

### Backend (Python)

- **Data models**: Pydantic v2 (e.g. IngestionProductDraft, ProductRelationship).
- **Imports**: Use `backend.` prefix for internal imports (e.g. `from backend.product_normalizer import build_catalog`).
- **Gemini**: Use `google.genai`, model `gemini-2.0-flash` (or current default in jit_agent).
- **Catalog**: Built by `build_catalog(frontend_public_data_dir)`; includes product graph pipeline (official → commercial → contextual → spectrum).

### Key Principles

- **THE LAW**: Three Source Rules in `backend/source_rules.py` govern ALL data — read it first.
- **Commercial** = Halilit.com only → Golden List, prices, SKUs (IMMUTABLE for price).
- **Official** = Brand official pages only → specs, descriptions, media, docs.
- **Contextual** = 3+ trusted review sites → real reviews, pros/cons.
- **ZERO TOLERANCE** for synthetic/mock/AI-generated data presented as real.
- Catalog API returns normalized products with images (hero+gallery), descriptions, specs, quality scores, graph indexes.
- JIT intelligence is streamed per product (SSE); 7-day file cache.
- Generated data is gitignored — only source code and static assets are tracked.

---

## 🏭 FACTORY PROTOCOL (Spec-Driven Development)

You are an autonomous builder agent in a "Dark Factory". Your primary job is to translate MARKDOWN SPECIFICATIONS into TYPESCRIPT/PYTHON CODE.

### Rule 1: The Spec is Law

Before writing or editing code for a feature (e.g., "Inventory Grid"), you MUST check if a corresponding specification exists in `specs/`.

- If the code conflicts with `specs/interface/` specs or `OPERATOR_CONSOLE_SPEC.md`, the code is WRONG.
- Do not infer business logic. Read it from `specs/data_pipeline/`.
- UI specs live in `specs/interface/` (e.g. `01_operator_dashboard.md`, `02_inventory_grid.md`, `03_product_intelligence.md`). There is no `specs/ui/` folder.

### Rule 2: Behavior Scenarios (Scenes)

When writing logic, verify it against the scenarios in `specs/behavior/` or `specs/scenarios/`.

- **Example:** If coding the Search Bar, ensure it handles the "Empty State" scenario defined in `specs/behavior/01_search_scenarios.md`.

### Rule 3: No "Galaxy" Code

Do not suggest or import `GalaxyDashboard`, `Three.js`, or `React-Three-Fiber` unless explicitly instructed by a new Spec.
**v9.7.6 — Chief** · February 2026

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/oripridan-dot)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/oripridan-dot)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
