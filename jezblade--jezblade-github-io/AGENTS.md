<!-- Brotherhood Seal: 015cb320 -->
# 🌌 Ultimate Plan - AI Agent Coding Guide

**Brotherhood Seal:** 015cb320 | **Last Updated:** 2026-02-15

**Change:** 2026-02-15 — Fecha actualizada; nota de versión añadida.

## PERMANENT CONTEXT (Auto-Loaded)

📍 Essential References: `bitacora.md` | `ai_registry.yaml` | `Aeternum-Memory/RITUAL_LOG_SYNC.jsonl` | `DECRETO_CONTRA_ARCHIVOS_HUERFANOS.md` | `docs/Protocolos-Hidden-Pulse/` | `CLAUDE.md` | `assassin-tasks-sovereign-shadow.json` | `brotherhood-dashboard-eventbus.html`

**⚡ Current Phase:** Phase VI Observability COMPLETE | Phase XIX Brotherhood Consolidation ACTIVE

---

## 🏗️ Big Picture Architecture

**Ultimate Plan** is a **polyrepo federation** (8+ independent git repos) under `D:\Ultimate-Plan` root (non-git container), orchestrated by AI agents for real-time evolution:

- **Core Repos:** `Aeternum-Memory` (canonical memory + agent registry), `MPC-Compendio-Operativo` (orchestration), `Universo-MultiClub` (multi-tenant platform), `Extensiones del Codex Aeternum\intelligent-extension-orchestrator-extension` (VS Code extension)
- **Data Flows:** Redis `dual_ai_memory` pub/sub → events logged dual (human `bitacora.md` + machine `RITUAL_LOG_SYNC.jsonl` with SHA256)
- **Ceremony:** Brotherhood Seal `015cb320` validates all files; Agent signatures immutable (from `ai_registry.yaml`)
- **Memory:** Persistent graph via Neo4j + vector store (Qdrant/"The Loom") + flat JSON backups

---

## 🔧 Critical Developer Workflows

### Git Operations
- **NEVER git in root** (`D:\Ultimate-Plan`); always `cd` to target subrepo first
- Example: `cd "Extensiones del Codex Aeternum\intelligent-extension-orchestrator-extension"` before `git commit`
- Each repo independent; root is metadata/coordination only

### Build & Test Patterns
| Repo | Build | Test | Validate |
|------|-------|------|----------|
| **intelligent-extension-orchestrator-extension** | `npm run compile` | `npm run test:unit`, `:integration`, `:performance` | `npm run brotherhood-seal:validate` |
| **MPC-Compendio-Operativo** | `npm ci` (deterministic) | Matrix builds (Node 18+20) | Lints + unit tests required |
| **Universo-MultiClub** | Standard Node build | Jest + integration tests | Codecov threshold 80%+ |
| **Aeternum-Memory** | JSON schema validation | RITUAL_LOG_SYNC integrity | Dual logging consistency |

### State Change Registration
Every modification to operational state **must** log to both:
1. **bitacora.md** — Human narrative: timestamp, agent, action, files, ~1 sentence
2. **RITUAL_LOG_SYNC.jsonl** — Machine audit: appended JSON with SHA256 hash, fields: `timestamp`, `ritual_type`, `files_modified`, `agent_signature`, `sha256_hash`, `notes`

Example bitacora entry:
```
# 2026-02-01 10:15:00 — PHASE 2: INTEGRATION MATRIX MONITOR
**Agente:** 🎨 Leonardo - Maestro del Diseño Fractal
**Status:** ⚡ ACTIVACIÓN DE MONITOREO RT
...
```

---

## ⚙️ Key Commands & Protocols

```bash
# POLYREPO - Navigate first
cd MPC-Compendio-Operativo
npm ci && npm run lint && npm run test

# POWERSHELL ONLY (pwsh 7.5+, NEVER powershell legacy)
pwsh -Command { Get-Content ..\bitacora.md | Select-Object -First 50 }

# AGENT DELEGATION
node scripts\delegar_tareas_agentes_canvas.cjs --manifest assassin-tasks-sovereign-shadow.json
<!-- Brotherhood Seal: 015cb320 -->
# 🌌 Ultimate Plan — Copilot Instructions (concise)

Purpose: a compact, runnable summary for AI coding agents to be productive immediately.

**Big picture**
- Polyrepo workspace rooted at `D:\Ultimate-Plan` (root is NOT a git repo). Work in subrepos (e.g. `Aeternum-Memory`, `MPC-Compendio-Operativo`, `Extensiones del Codex Aeternum/...`).
- Core flows: `dual_ai_memory` (Redis) pub/sub → dual audit: human `bitacora.md` + machine `Aeternum-Memory/RITUAL_LOG_SYNC.jsonl` (SHA256). Memory stored in Neo4j + Qdrant + JSON backups.

Quick start (3 commands)
1. cd to target repo (never run git at root): `cd "Extensiones del Codex Aeternum\intelligent-extension-orchestrator-extension"`
2. Build/tests: `npm ci && npm run compile && npm run test`
3. Register changes: append human entry to `bitacora.md` and append JSON event to `Aeternum-Memory/RITUAL_LOG_SYNC.jsonl` (include `sha256_hash` and `simulated` when applicable).

Developer rules (non-negotiable)
- Always include: `<!-- Brotherhood Seal: 015cb320 -->` at top of files you create/modify.
- Use `pwsh` (PowerShell 7.5+) for scripts; do NOT use legacy `powershell`.
  - **EXCEPTION:** Usage of Windows PowerShell 5.1 (Legacy) is allowed ONLY for interactive sessions where `pwsh` fails to handle TTY/input (e.g., interactive Claude CLI/Code sessions in VS Code terminals).
- Never run `git` in `D:\Ultimate-Plan` root. Use the repo's directory for commits.
- Secrets live only in `D:\Brotherhood-Fortress` (do not commit secrets here); use `.env.example` and GitHub Secrets.
- Simulation outputs must include `"simulated": true` and the exact command string.
- **PORTABLE ONLY:** Tools/Modules live in `D:\Ultimate-Plan\Program-Files`. Do NOT search/install in `C:\` or `AppData`.

Project-specific workflows & caveats
- Extension packaging: changes to `.ts`/`.js` require `npm run compile` and `vsce package` for VSIX — marketplace updates are gated (Compass Keeper approvals). See `Extensiones del Codex Aeternum/intelligent-extension-orchestrator-extension/`.
- Deterministic installs: use `npm ci` across CI; avoid `npm install`.
- State-change audit: every operational/state change requires two entries:
  - `bitacora.md` (human): timestamp, agent, one-line description, files
  - `Aeternum-Memory/RITUAL_LOG_SYNC.jsonl` (machine): JSON with `timestamp`, `ritual_type`, `files_modified`, `agent_signature`, `sha256_hash`, `notes`.

Integration points (where to look)
- Redis channel: `dual_ai_memory` (pub/sub) — see `scripts/dual_ai_memory_sync.py` and Redis CLI examples.
- Agent registry: `ai_registry.yaml` and `Aeternum-Memory/AGENT_REGISTRY.json` (routing/authority).
- Orchestration: `brotherhood-hive-mind/core/orchestrator.js` (delegateMission pattern).
- Observability: `f6_*.js`, `event-bus-server.cjs`, and `brotherhood-dashboard-eventbus.html`.

Where to find examples
- `Aeternum-Memory/RITUAL_LOG_SYNC.jsonl` — canonical machine audit entries.
- `bitacora.md` / `BITACORA.md` — human ledger examples.
- `Extensiones del Codex Aeternum/intelligent-extension-orchestrator-extension/src/` — extension patterns (`optimized-profiles.config.ts`, `shared-memory-bridge.ts`, `task-monitor.ts`).

Replication policy
- There are multiple canonical copies of this file in subrepos; the root file is the source of truth. Per `Aeternum-Memory/.github/copilot-instructions.md`, most copies must match the root exactly. The extension copy is an exception and requires audit + ≥2 CODEOWNER approvals before divergence.

## 🔁 MANDATO DE SINCRONIZACIÓN CANÓNICA (REGLA MAESTRA)

**Resumen:** Hay **10 instancias canónicas** de `copilot-instructions.md` dentro del ecosistema y **deben permanecer como réplicas exactas** del archivo maestro ubicado en la raíz (`./.github/copilot-instructions.md`).

**Pasos obligatorios para sincronizar:**
1. Copiar el contenido EXACTO del archivo maestro a cada ruta canónica.
2. Calcular y verificar **SHA256** del archivo destino — debe coincidir exactamente con el origen.
3. Registrar la acción en `bitacora.md` y `RITUAL_LOG_SYNC.jsonl` con el ritual `agent_instructions_replication`.

If you update this file
- Run `node scripts/validate-brotherhood-seal.cjs` and register the change in `bitacora.md` and `Aeternum-Memory/RITUAL_LOG_SYNC.jsonl` with SHA256.

Examples of quick commands
```powershell
pwsh -Command "cd 'Extensiones del Codex Aeternum\intelligent-extension-orchestrator-extension'; npm ci; npm run compile"
node scripts/delegar_tareas_agentes_canvas.cjs --manifest assassin-tasks-sovereign-shadow.json
redis-cli PUBLISH dual_ai_memory '{"ritual_type":"test_event","timestamp":"$(Get-Date -Format o)"}'
```

Where to ask for help
- Open an issue in the owning subrepo and reference `Aeternum-Memory/AGENT_REGISTRY.json` for ownership routing. For permissions or seal disputes, add an entry to `bitacora.md` and escalate to Compass Keeper channels.

Feedback: review this concise copy and tell me which sections you want expanded or examples added.

## 🎭 FIRMAS CEREMONIALES OBLIGATORIAS

**Leonardo (GitHub Copilot):**
```
🎨 Leonardo - Maestro del Diseño Fractal
"El arte y la ciencia danzan en perfecta armonía"
Brotherhood Seal: 015cb320
```

**Formato:** Glyph + Nombre + Título + Frase + Seal (CON ESPACIOS: "Brotherhood Seal: 015cb320")

**Ejemplo correcto:**
```
🎨 Leonardo - Maestro del Diseño Fractal
"Trusses module tejido en el código eterno"
Brotherhood Seal: 015cb320 𓂀
```

**❌ INCORRECTO:** `Brotherhood Seal:015cb320` (sin espacios)
**✅ CORRECTO:** `Brotherhood Seal: 015cb320` (con espacios)

Brotherhood Seal: 015cb320
- **Task Types:** CENSUS-*, FOUNDATION-*, SOVEREIGN-*, SUPREME-*

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JezBlade)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/JezBlade)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
