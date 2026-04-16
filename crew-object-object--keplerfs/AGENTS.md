# Copilot Instructions — KeplerFS

## Project Overview

KeplerFS is a distributed, peer-to-peer file system. The monorepo contains:

- **`packages/node`** — Bun TypeScript binary. Every peer runs this same binary as HTTP gateway + gossip peer + chunk store. No master node.
- **`apps/frontend`** — SvelteKit 5 + Tailwind CSS 4 web UI for upload, download, and cluster visualisation.

---

## Tech Stack

| Layer                     | Technology                              |
| ------------------------- | --------------------------------------- |
| Node runtime              | [Bun](https://bun.sh) ≥ 1.2             |
| Node HTTP                 | `Bun.serve` with typed routes           |
| Node TCP (chunk transfer) | `Bun.listen` / `Bun.connect`            |
| Node UDP (gossip)         | `Bun.udpSocket`                         |
| Node persistence          | `bun:sqlite` (Database)                 |
| Frontend framework        | SvelteKit 5 (runes mode)                |
| Frontend styling          | Tailwind CSS 4 + `tw-animate-css`       |
| Frontend components       | shadcn-svelte (in `$lib/components/ui`) |
| Package manager           | pnpm workspaces                         |
| Language                  | TypeScript strict mode throughout       |

---

## Architecture Invariants

When touching `packages/node`, keep these invariants:

1. **Content addressing** — every chunk is identified by its SHA-256 hex digest. Never refer to a chunk by path alone.
2. **Atomic writes** — always write chunks to `<path>.tmp`, call `fsync`, then `rename`. Never mark a chunk as available before this sequence completes.
3. **Ring-first ownership** — ownership of a chunk is determined by the consistent hash ring (`src/ring/index.ts`). Never hard-code which node owns what.
4. **Replication queue durability** — any chunk that must be sent to a remote node must first be persisted in `replication_queue` before the background worker picks it up. This enables at-least-once delivery with crash recovery.
5. **No master** — there is no coordinator. All cluster state (peer tables, file manifests) is propagated via gossip. Code must not assume a special "primary" node exists.

---

## Code Conventions

### General

- Use TypeScript `strict` mode. All function signatures must be explicitly typed — no implicit `any`.
- Prefer `const` over `let`. Use `let` only when reassignment is needed.
- Prefer named exports over default exports (except SvelteKit routes/layouts).
- Never use `process.exit()` directly — throw structured errors that propagate to the main entry point.
- Use `crypto.createHash("sha256")` for hashing; `crypto.randomUUID()` for IDs.

### Bun-specific

- Use `Bun.file()` + `.arrayBuffer()` / `.stream()` for file I/O, not `fs.readFile`.
- Use `bun:sqlite` for all persistence. Never introduce an external ORM.
- Prepared statements should be cached on the `Database` instance via `db.query(sql)`.
- For TCP framing, use the 8-byte header format defined in `src/transfer/protocol.ts`.
- For gossip messages, use the `GossipMessage` union type in `src/gossip/types.ts`.

### SvelteKit / Frontend

- Use Svelte 5 runes (`$state`, `$derived`, `$effect`, `$props`) — no legacy `writable` stores.
- All API calls go through `src/lib/api.ts`; no raw `fetch` in components.
- Icons: use `@lucide/svelte` only.
- shadcn-svelte components live in `$lib/components/ui`. Do not modify generated component files.

---

## File Layout — `packages/node/src`

```
config.ts               env var → NodeConfig
types.ts                all shared interfaces
db/
  schema.ts             SQL DDL strings
  index.ts              Database singleton + init
ring/
  index.ts              ConsistentHashRing class
gossip/
  types.ts              GossipMessage union
  index.ts              GossipManager class (UDP)
storage/
  index.ts              ChunkStore class (atomic R/W/delete)
transfer/
  protocol.ts           wire framing helpers
  server.ts             TCP server — receives chunks from peers
  client.ts             TCP client — sends chunks to peers
replication/
  queue.ts              ReplicationQueue (SQLite-backed)
  pipeline.ts           ReplicationPipeline (background worker)
http/
  routes/
    files.ts            POST /file, GET /file/:id
    peers.ts            GET /peers
    health.ts           GET /health
  index.ts              Bun.serve wiring
index.ts                startup + shutdown sequence
```

---

## Testing

- Tests live alongside source files as `*.test.ts`.
- Run with `bun test` from `packages/node`.
- Unit-test pure functions (ring, hashing, framing) without I/O mocks.
- Integration tests for HTTP routes use `new Request(...)` + call the handler directly.

---

## Common Pitfalls

- **Don't use `setTimeout` for gossip timers** — use `setInterval` and keep a reference so it can be cleared on shutdown.
- **Don't stream chunks over HTTP** between nodes — use the dedicated TCP transfer layer.
- **Don't read a whole file into memory to hash it** — use a streaming SHA-256 over the `ReadableStream` of the chunk.
- **Always check ring membership** before deciding a chunk is "local" — the ring can change as nodes join/leave.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Crew-object-Object) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
