# Copilot Instructions — Graph Visualizer

## Architecture

Monorepo with two independent apps communicating via REST + WebSocket:

- **`backend-nodejs/`** — Express + TypeScript API (port 8080). Multi-engine graph DB abstraction (Neo4j, Memgraph, MSSQL, ArangoDB). Engine selected per-request via `?engine=` query param.
- **`frontend-graph-viewer/`** — React 18 + Vite + TypeScript SPA (port 5173). 10+ interchangeable views (7 graph viewers + impact analysis + query panel + algorithm panel + benchmark). All state in `App.tsx` via 18 `useState` hooks — no Redux/Zustand.

## Dev Workflow

```bash
cd backend-nodejs && npm run dev      # tsx watch, port 8080
cd frontend-graph-viewer && npm run dev  # Vite, port 5173
```

- `npm run typecheck` (backend) — `tsc --noEmit`
- **No tests exist** — `npm test` is a no-op placeholder in both packages
- **No .eslintrc/.prettierrc** — linting script exists but no config file
- Bulk graph generation: `cd backend-nodejs && node create-engine-graphs.mjs --engine=neo4j`

## Backend Patterns

### Module System

- **ESM only** (`"type": "module"`). All `.ts` imports use **`.js` extensions**:
  ```typescript
  import { Neo4jService } from "./services/Neo4jService.js";
  ```
- TypeScript strict mode, target ES2022, output to `dist/`

### Multi-Engine Strategy

All engines implement `GraphDatabaseService` interface (`src/services/GraphDatabaseService.ts`). Engines registered conditionally in `src/index.ts` based on env vars (`NEO4J_URI`, `MEMGRAPH_URI`, `MSSQL_HOST`, `ARANGO_URL`). The `resolveEngine` middleware reads `?engine=` and sets `(req as any).dbService`.

- `MemgraphService` **extends** `Neo4jService` — overrides driver + `computeImpact` only
- `neo4j-driver-memgraph` is an **npm alias** (`"neo4j-driver-memgraph": "npm:neo4j-driver@^4.4.11"`)
- Neo4j driver must be v5.28.3 (v6 breaks). Memgraph uses v4 via the alias

### Route Structure

| File | Mount | Key endpoints |
|------|-------|---------------|
| `graphRoutes.ts` | `/api` | CRUD graphs, impact, neighbors, benchmark |
| `databaseRoutes.ts` | `/api/databases` | List/create/delete databases |
| `algorithmRoutes.ts` | `/api` | `POST /api/graphs/:id/algorithms` — 14 algorithms |
| `cmdbRoutes.ts` | `/api/cmdb` | EasyVista CMDB import (only if MSSQL active) |
| **Inline in index.ts** | `/api` | `POST /api/query`, `GET /api/health`, `GET /api/engines` |

Route files export **factory functions** receiving `(service, broadcast?)` → `Router`.

### AlgorithmService (`src/services/AlgorithmService.ts`)

993-line pure in-memory graph algorithm engine. Builds adjacency lists from `GraphData`, runs: BFS, DFS, Bidirectional BFS, Dijkstra, Degree/Betweenness/Closeness centrality, PageRank, Louvain, Label Propagation, Connected/Strongly Connected Components, Topological Sort, Cascading Failure simulation.

### WebSocket

`ws` server on `/ws`. Broadcasts `graph:created`, `graph:deleted` events. `broadcast()` passed to route factories.

### Caching

`NodeCache` with 5-min TTL. Key: `graph:<database>:<graphId>`. Bypass: `?nocache=true`. Stats: `GET /optim/cache/stats`. Compression bypass: `?nocompress=true`.

### Error Handling

Routes use `try/catch` → `next(error)`. Global handler in `index.ts` returns `{error: message}`. Logging is **inconsistent**: pino in `index.ts`, `console.log/error` in services.

### Engine-Specific Constraints

- **MSSQL**: 2100 param limit → batch 500 nodes / 400 edges. CTE `MAXRECURSION 200`. Lazy connection pools per database via `Map<string, ConnectionPool>`
- **Memgraph**: No `length(path)` (use `size(relationships(path))`), no multi-database, no composite indexes
- **Neo4j**: `bolt://` standalone, `neo4j://` cluster
- **ArangoDB**: `collection.import()` supports 5000/batch

### Adding a New Engine

1. Implement `GraphDatabaseService` in `src/services/`
2. Add conditional init in `src/index.ts` based on new env var
3. Register in the `engines` record

### Dead Code

- `src/config/database.ts` — legacy, never imported
- `zod` — in `package.json` dependencies but never imported

## Frontend Patterns

### Viewer Components (all in `src/components/`)

`ViewerType` union: `'force-graph' | '3d' | 'sigma' | 'g6' | 'd3' | 'cytoscape' | 'vis-network' | 'impact' | 'query' | 'algorithms' | 'benchmark'`

- **Two data paths**: `GraphViewer` receives pre-transformed `ForceGraphData`; all others receive raw `GraphData`
- **Adaptive rendering**: viewers adjust labels/physics/sizes by `nodeCount` thresholds (<500, 500–2k, 2k–5k, 5k–10k, >10k)
- `SigmaGraphViewer.tsx` (1689 lines) is the largest — progressive loading with node list panel, `exploreNode()`, ForceAtlas2, position caching in localStorage via `nodePositionCache.ts`

### State Flow in App.tsx

Cascading `useEffect` chain (order matters):
1. Mount → `loadEngines()` → sets `selectedEngine`
2. `[selectedEngine]` → `loadDatabases()` → sets `selectedDatabase`
3. `[selectedDatabase]` → `loadGraphs()` → auto-selects first graph

Cross-component: `OptimPanel` uses `window.__optimSetLastLoad` global callback (not props/context).

### API Client (`src/services/api.ts`)

Axios client at `http://127.0.0.1:8080/api`. 6 namespaces: `graphApi`, `cmdbApi`, `databaseApi`, `optimApi`, `engineApi`, `algorithmApi`. All methods accept optional `database` and `engine` query params. `getGraph()` extracts perf headers (`X-Cache`, `X-Response-Time`, etc.) into `GraphLoadResult`.

### Hooks

- `useTheme` — dark/light toggle via localStorage + `data-theme` attribute
- `useWebSocket` — auto-reconnecting WS to `ws://127.0.0.1:8080/ws`, 3s reconnect

### Conventions

- **No path aliases** — all imports are relative, no barrel files
- **Color system** in `graphTransform.ts`: 30+ `NODE_COLORS` + deterministic HSL hash fallback
- **CSS co-location**: each component has a paired `.css` file
- Frontend imports can use `.tsx` extensions (`allowImportingTsExtensions: true`)

## Data Model

Shared types (`backend: src/models/graph.ts` / `frontend: src/types/graph.ts`):

- `GraphNode` — `{id, label, node_type, properties}`
- `GraphEdge` — `{source, target, label?, edge_type, properties}`
- `GraphData` — `{nodes[], edges[]}`
- `GraphSummary` — list metadata with `node_count`, `edge_count`
- `ImpactResult` — `{sourceNodeId, impactedNodes[{nodeId, level}], depth, elapsed_ms, engine}`

Algorithm results use a discriminated union: `TraversalResult | ShortestPathResult | CentralityResult | CommunityResult | TopologicalSortResult`

## Environment Variables

Backend env vars in `.env` (comment/uncomment to enable/disable engines):

| Variable | Purpose | Required |
|----------|---------|----------|
| `NEO4J_URI` | `bolt://host:7687` | At least one engine |
| `MEMGRAPH_URI` | `bolt://host:7688` (no auth) | must be active |
| `MSSQL_HOST/PORT/USER/PASSWORD/DATABASE` | SQL Server | |
| `ARANGO_URL` | `http://host:8529` | |
| `SERVER_PORT` | API port (default 8080) | No |
| `SERVER_HOST` | Bind address (default 127.0.0.1) | No |
| `LOG_LEVEL` | pino level (default "info") | No |
| `DEFAULT_ENGINE` | Auto-detected if unset | No |

Request size limit: `express.json({ limit: "50mb" })`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Aurel-22) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
