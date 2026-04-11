---
description: Implementiere einen produktionsreifen Go Port Summary Service mit gRPC, Event-Driven Updates und Python-Integration.
---

# 🎯 Ziel

Baue einen **Go-basierten Port Summary Service** (Port 50054), der Geräte-, Interface- und Link-Informationen performant zusammenfasst und über gRPC bereitstellt.  
Der Service muss **Production-Grade** sein, skalieren auf 10.000+ Devices und Event-Driven Updates aus PostgreSQL verarbeiten.

---

# 📋 Anforderungen

## Backend (Go Service)

- Implementiere `port_summary.proto` mit Methoden:
  - `GetPortSummary(deviceID)` → `InterfaceSummary[]`
  - `GetBulkPortSummary(deviceIDs[])` → `map[id]summary`
  - `InvalidateCache(deviceID)`
- In-Memory State:
  - `devices`, `interfaces`, `links`
  - `ponOccupancy` (oltID → ponIfID → count)
  - `opticalPaths` (ontID → ponIfID, precomputed)
- Event Listener (PostgreSQL NOTIFY):
  - `link_created/deleted` → recompute OLT
  - `device_provisioned` → update optical path
  - `topology_version_change` → full reload
- Thread-Safe (RWMutex)

## Python Integration

- gRPC Client: `backend/clients/port_summary_client.py`
- API Endpoint: `/api/ports/summary`
- Fallback-Logik (wie Traffic Engine)

## Performance-Ziele

- 70 Devices: 5–10ms (statt 250–700ms)
- 200 Devices: 10–20ms (statt 2–5s)
- 1000 Devices: 20–50ms (statt 10–30s)
- Skalierbar auf 10.000+ Devices

---

# 🏗 Architektur

Go Service (Port 50054) ── gRPC ── FastAPI Backend ── REST /api/ports/summary │ ▼ PostgreSQL (Events: link, device, topology)

---

# 📅 Zeitplan (Week 3)

- **Day 18**: Core Service
  - Proto Definition
  - Database Loader
  - Counting Logic
  - gRPC Server
- **Day 19**: Event Integration
  - PostgreSQL Listener
  - Cache Invalidation
  - Optical Path Precomputation
  - Integration Tests
- **Day 20**: Python Integration
  - gRPC Client
  - API Migration
  - E2E Tests

---

# 🧪 Tests

- Unit-Tests für Counting Logic
- Integrationstests (gRPC Client ↔ Go Service)
- E2E Tests (Python → Go → DB)
- Performance-Benchmarks (70, 200, 1000 Devices)

---

# 🚧 Risiken & Mitigation

- **Optical Path Complexity** → Reuse PathFinder Service
- **Memory Usage** → ~100MB bei 10k Devices, akzeptabel
- **Event Race Conditions** → Mutex + Event Queue
- **Integration Issues** → Copy Traffic Engine Client Pattern

---

# 🚀 Aktionen für den Agenten

- `#edit` → Go Service implementieren
- `#fetch` → Proto & DB-Schema laden
- `#test` → Unit- und Integrationstests ausführen
- `#run` → Performance-Benchmarks prüfen
- Roadmap & Tasks automatisch aktualisieren

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Duly330AI)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Duly330AI)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
