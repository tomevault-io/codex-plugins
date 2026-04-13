# Project Guidelines

## Projektbeschreibung / Descripción del proyecto

Interaktive Demo des Microsoft Agent Framework — ein Python-SDK für KI-gesteuerte Multi-Agenten-Workflows mit React-Frontend zur Visualisierung.

Demo interactiva del Microsoft Agent Framework — un SDK de Python para workflows multi-agente con IA, con frontend React para visualización.

## Sprache / Idioma

- Alle UI-Texte und Dokumentation: **Deutsch und Spanisch** (bilingual).
- Codekommentare und Variablennamen: **Englisch**.
- Toda la UI y documentación: **alemán y español** (bilingüe).
- Comentarios y nombres de variables en código: **inglés**.

## Architektur / Arquitectura

```
Frontend:  React + Vite + Tailwind CSS 4 + Motion (Framer Motion)
Backend:   Python mit Microsoft Agent Framework SDK
LLM:       Ollama (lokal) oder OpenAI GPT-4o (Cloud)
RAG:       ChromaDB + sentence-transformers + PyMuPDF
```

- `src/App.tsx` — Haupt-React-Komponente mit drei Tabs (Support-Workflow, Corporate Triage, Quellcode)
- `agent_workflow.py` — Multi-Agent-Graph: Triage → TechSupport oder Refund mit HITL
- `corporate_support.py` — Einzelner Agent mit strukturierter Ausgabe und Tool-Genehmigung

## Code Style

- TypeScript mit React funktionalen Komponenten und Hooks.
- Python: async/await, Pydantic-Modelle für strukturierte Ausgaben, Type Hints.
- Tailwind CSS direkt im JSX — keine separaten CSS-Dateien außer `index.css` für globale Stile.
- Farben: Slate/Amber Corporate-Theme (hell, nicht dunkel).

## Build & Run

```bash
# Frontend
npm install
npm run dev          # Vite auf Port 3000

# Python Backend
pip install -r requirements.txt
python agent_workflow.py
python corporate_support.py
```

## UNC-Pfad-Hinweis / Nota sobre rutas UNC

Der Workspace liegt auf einem Netzwerkpfad (`\\smedfa\...`). Für Git und Vite muss nach `C:\temp\agent-demo` kopiert werden. npm benötigt `$env:npm_config_script_shell = pwsh`.

El workspace está en una ruta de red UNC. Para Git y Vite hay que copiar a `C:\temp\agent-demo`. npm necesita `$env:npm_config_script_shell = pwsh`.

## Konventionen / Convenciones

- Demo-Projekt: keine Produktions-Optimierungen, kein Over-Engineering.
- Preset-Beispiele als Array-Konstanten (`demoBeispiele`, `corporateBeispiele`).
- HITL-Flows: `ctx.request_info()` für menschliche Genehmigung, `approval_mode="always_require"` für Tool-Freigabe.
- Neue Dokumentation immer bilingual (DE + ES) erstellen.
- Proyecto demo: sin optimizaciones de producción, sin sobre-ingeniería.
- Ejemplos preset como constantes de array.
- Documentación nueva siempre bilingüe (DE + ES).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MiguelColonia)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/MiguelColonia)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
