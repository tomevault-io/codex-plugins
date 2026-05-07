# A2Z DSA Sheet - AI Agent Instructions

## Project Overview
This project is a personal Data Structures and Algorithms (DSA) tracker and roadmap viewer. It consists of a static frontend (`index.html`) that renders a structured curriculum from a JSON data file (`a2z.json`), supported by Python scripts for data maintenance and content archiving.

## Architecture & Data Flow
- **Data Source**: `a2z.json` is the single source of truth. It defines the hierarchical structure:
  - **Steps** (Broad categories like "Learn the basics")
  - **Sub-steps** (Specific modules like "Things to Know in C++")
  - **Topics** (Individual problems/concepts with links to resources)
- **Frontend**: `index.html` is a vanilla HTML/JS Single Page Application. It fetches `a2z.json` at runtime to generate the UI dynamically.
- **Maintenance**: Python scripts in `scripts/` perform ETL operations on `a2z.json` (cleaning URLs, expanding shortlinks) or consume it (downloading articles).

## Critical Workflows

### 1. Running the Frontend
Because `index.html` uses `fetch('a2z.json')`, it may encounter CORS issues if opened directly via `file://` protocol.
- **Recommended**: Serve locally using Python.
  ```bash
  python3 -m http.server
  # Then open http://localhost:8000
  ```

### 2. Data Maintenance Scripts
Scripts are standalone and typically operate on `a2z.json`.
- **Clean Tracking Params**: Removes `utm_`, `fbclid`, etc.
  ```bash
  python3 scripts/clean_trackers.py a2z.json
  ```
- **Expand Shortlinks**: Resolves `bit.ly` links to their final destination.
  ```bash
  python3 scripts/debitlify.py a2z.json
  ```
- **Archive Content**: Downloads linked articles to local `articles/` directory.
  ```bash
  python3 scripts/download_articles.py
  ```

## Key Conventions & Patterns

### JSON Schema (`a2z.json`)
When modifying the data file, strictly adhere to this hierarchy:
```json
[
  {
    "step_no": 1,
    "step_title": "...",
    "sub_steps": [
      {
        "sub_step_no": 1,
        "sub_step_title": "...",
        "topics": [
          {
            "id": "unique_string_id",
            "question_title": "...",
            "post_link": "...",
            "yt_link": "...",
            "plus_link": "..."
          }
        ]
      }
    ]
  }
]
```

### Python Scripts
- Scripts should be robust against network failures (timeouts, retries).
- Use `sys.argv` for file input arguments.
- When processing URLs, handle edge cases like missing schemes or trailing slashes.

### Frontend (index.html)
- Vanilla JS, no build step required.
- CSS variables are used for theming (defined in `:root`).
- DOM manipulation is done directly; ensure element IDs match the logic.

---
> Source: [Bugaddr/a2z_old_sheet](https://github.com/Bugaddr/a2z_old_sheet) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-07 -->
