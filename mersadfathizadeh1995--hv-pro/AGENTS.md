# HVSR Pro — Copilot Custom Instructions

This repository contains **HV Pro**, a PyQt5 desktop application and MCP-enabled API
for Horizontal-to-Vertical Spectral Ratio (HVSR) seismic analysis.

## MCP Server

The `hvsr-pro` MCP server exposes the full HVSR analysis pipeline as tools.
See `.github/skills/hvsr-analysis.md` for the complete skill reference.

## Codebase Conventions

- **Branch:** feature work happens on `api_layer`; `main` is the stable GUI release.
- **API layer:** `hvsr_pro/api/` — config-driven, headless pipeline used by both GUI and MCP.
- **GUI:** `hvsr_pro/gui/` — PyQt5 frontend that delegates to the API layer.
- **MCP:** `hvsr_pro/mcp/` — FastMCP server wrapping the API for AI agent access.
- **Statistics default:** lognormal (log-space median ± percentiles). Never use arithmetic mean as default.
- **Peak basis default:** median curve (not mean).
- **Time range:** always provide `timezone_offset` when working with local times. Data internally uses UTC.
- **File paths:** use absolute paths with backslashes on Windows.

---
> Source: [mersadfathizadeh1995/HV_Pro](https://github.com/mersadfathizadeh1995/HV_Pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-12 -->
