# GraphFit — Master Project Reference

> **Read this file first.** It is the single entry point for understanding the entire project.
> For deeper context, follow the links to the specialized docs below.

---

## Project Overview

GraphFit is a web app where users upload a **photo of a data table**, the system OCRs the numbers, fits a curve, and displays the fitted graph + equation. It supports 11 physics/chemistry experiment modes. Deployed on Render.

## Data Flow

```
Home (select mode) → Upload (image) → POST /api/extract (OCR) → Review (editable table)
    → POST /api/fit (curve fitting + matplotlib) → Results (graph + equation)
```

## Tech Stack

| Layer    | Stack                                                         |
|----------|---------------------------------------------------------------|
| Frontend | React 19, Vite 6, Tailwind CSS v4, react-router-dom v7       |
| Backend  | FastAPI (Python 3.11), numpy, scipy, matplotlib               |
| OCR      | Groq API — `meta-llama/llama-4-scout-17b-16e-instruct`       |
| Deploy   | Render (render.yaml) — single web service, gunicorn + uvicorn |
| State    | React Context (no Redux), no database (stateless)             |

## Documentation Map

| File | What it covers | When to read it |
|------|---------------|-----------------|
| **claude.md** (this file) | Project overview, architecture, commands, conventions | Always — start here |
| **frontend.md** | Frontend architecture, routes, components, state, user flow | Working on React UI |
| **backend.md** | Backend services, fitting functions, plotting, OCR details | Working on Python backend |
| **MENU_ITEM_SPEC.md** | Step-by-step checklist + examples for adding new fitting modes | Adding a new experiment |
| **.github/copilot-instructions.md** | Concise quick-reference (auto-loaded by GitHub Copilot) | Auto-loaded, no need to open |
| **product_description.md** | Original product brief / vision doc | Understanding the "why" |

## Repo Structure

```
graph_figure_out/
├── claude.md                    # ← YOU ARE HERE — master reference
├── frontend.md                  # Frontend deep-dive
├── backend.md                   # Backend deep-dive
├── MENU_ITEM_SPEC.md            # Adding new modes checklist
├── product_description.md       # Original product brief
├── render.yaml                  # Render deployment config
├── build.sh                     # Production build script
│
├── backend/
│   ├── main.py                  # FastAPI app, CORS, static serving
│   ├── requirements.txt         # Python deps
│   ├── routes/
│   │   ├── extract.py           # POST /api/extract — OCR endpoint
│   │   └── fit.py               # POST /api/fit, POST /api/fit-waves
│   ├── services/
│   │   ├── ocr.py               # Groq Vision OCR — per-mode column hints
│   │   ├── fitting.py           # All fit_*() functions (~600 lines)
│   │   └── plotting.py          # generate_graph() per-mode branches (~420 lines)
│   └── static/                  # Built frontend (created by build.sh)
│
└── frontend/
    ├── vite.config.js           # Dev proxy: /api → localhost:8000
    └── src/
        ├── App.jsx              # Router: / → /upload → /review → /results
        ├── context/AppContext.jsx   # Global state
        ├── components/          # Navbar, Layout, FileDropzone
        └── pages/
            ├── HomePage.jsx     # Mode selection (11 cards)
            ├── UploadPage.jsx   # Image upload + extract trigger
            ├── ReviewPage.jsx   # Editable data table
            └── ResultsPage.jsx  # Graph + equation display
```

## API Endpoints

| Endpoint | Method | Input | Output |
|----------|--------|-------|--------|
| `/api/extract` | POST | Multipart: `image` + `mode` | `{ columns, rows }` |
| `/api/fit` | POST | JSON: `{ mode, points, columns }` | `{ equation, graphImage, fitParams }` |
| `/api/fit-waves` | POST | JSON: `{ rope_points, rope_columns, sound_points, sound_columns }` | Dual graphs + fit params |
| `/api/health` | GET | — | `{ status: "ok" }` |

## Fitting Modes (11 total)

| ID | Title | Fit Method |
|----|-------|------------|
| `straight-line` | Straight Line | Linear regression |
| `cmc` | Chemistry — CMC | Piecewise breakpoint |
| `photoelectric-1-1` | Photoelectric V-I | Multi-series linear |
| `photoelectric-1-2` | Photoelectric (h) | Linear (V₀ vs ν) |
| `photoelectric-1-3` | Photoelectric Intensity | Multi-series linear |
| `single-slit` | Single Slit Diffraction | Sinc² fit |
| `newtons-rings` | Newton's Rings | Linear (r² vs m) |
| `pohls-damped` | Pohl's Pendulum (Damped) | Exponential decay |
| `pohls-forced` | Pohl's Pendulum (Forced) | Resonance curve |
| `polarization` | Optical Rotation | Linear regression |
| `waves` | Transverse & Longitudinal Waves | Dual: rope (3 groups) + sound |

## Adding a New Mode — Quick Reference

Touch these 5 files (see `MENU_ITEM_SPEC.md` for full details):

1. `frontend/src/pages/HomePage.jsx` → `FITTING_MODES` array
2. `backend/services/ocr.py` → `elif mode == "<id>":` with column hint
3. `backend/services/fitting.py` → `fit_<mode>()` returning `equation`, `description`, `r_squared`
4. `backend/routes/fit.py` → `elif req.mode == "<id>":` dispatch
5. `backend/services/plotting.py` → `elif mode == "<id>":` plot branch

## Conventions

- **Mode IDs**: kebab-case (`"straight-line"`, `"photoelectric-1-1"`)
- **Fitting functions**: `fit_<snake_case>()` in `services/fitting.py`
- **Frontend**: JSX, functional components, Tailwind utilities, React Context
- **Multi-series modes** (3+ columns): listed in `MULTI_SERIES_MODES` set in `routes/fit.py`
- **Waves mode** is special: separate endpoint `/api/fit-waves`, dual state in AppContext, conditional rendering in Upload/Review/Results pages

## Commands

```bash
# Frontend dev
cd frontend && npm run dev            # localhost:5173, proxies /api → :8000

# Backend dev
cd backend && uvicorn main:app --reload  # localhost:8000

# Production build (what Render runs)
chmod +x build.sh && ./build.sh        # installs deps, builds frontend → backend/static/
```

## Environment

- `GROQ_API_KEY` required in `backend/.env` or Render dashboard
- Python 3.11, Node 20
- No database — fully stateless request/response

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Propar2750) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
