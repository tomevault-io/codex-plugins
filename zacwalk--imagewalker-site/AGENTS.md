# Copilot Instructions

You are an expert software engineer who prioritizes **Theory Building** and **Design for Change**. Your goal is to write code that is not just syntactically correct, but creates a clear, shared mental model for both human developers and future AI agents.

## Project Overview

This is a legacy redirect site for **ImageWalker**, a Windows desktop photo management application. ImageWalker has been superseded by **Diffractor**. The site serves two purposes:
1. Redirect users to [diffractor.com](http://diffractor.com)
2. Provide downloads for archived older versions (2.01, 2.20, 2.31)

**Live URL:** imagewalker.com

## Tech Stack

- **Platform:** Google App Engine (Python 3.11 Standard Environment)
- **Framework:** Flask
- **Templating:** Jinja2 (via Flask)
- **Styling:** Modern CSS with custom properties (CSS variables)

## Project Structure

```
web/
├── app.yaml           # GAE configuration
├── main.py            # Flask application entry point
├── requirements.txt   # Python dependencies (Flask, gunicorn)
├── static/            # Static assets
│   ├── iw.css         # Main stylesheet
│   ├── diff.css       # Supplementary styles
│   ├── *.png          # Images (title, version screenshots, icons)
│   └── *.exe          # Archived installer downloads
└── templates/
    └── index.html     # Single-page Jinja2 template
```

## Design System

The site uses CSS custom properties for theming:
- `--accent`: Primary blue (#3b82f6)
- `--bg-primary/secondary/card`: Dark theme backgrounds
- `--text-primary/secondary/muted`: Text hierarchy
- `--radius-sm/md/lg`: Border radius tokens

## Local Development

```bash
cd web
python main.py
# Runs on http://localhost:8081
```

## Deployment

Deploy to Google App Engine:
```bash
cd web
gcloud app deploy
```

## Key Files

- [web/static/iw.css](web/static/iw.css) - Main styles with CSS variables and responsive design
- [web/templates/index.html](web/templates/index.html) - Single page template with Diffractor redirect CTA and version archive
- [web/main.py](web/main.py) - Flask routes with legacy static file serving


---
> Source: [ZacWalk/imagewalker-site](https://github.com/ZacWalk/imagewalker-site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-12 -->
