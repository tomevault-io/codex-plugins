# Math History Paper (LaTeX) — Copilot Instructions

This repository contains a LaTeX academic paper for the Tsinghua course **History of Mathematics (数学史)**.

## What you are editing

- The main entry file is `大一/正文/MathHistoryPaper/main.tex`.
- The body is modularized via `\input{sections/.../section}`.
- Bibliography is managed with **biblatex + biber** and stored in `大一/正文/MathHistoryPaper/references.bib`.

## Non-negotiable content rules

1. **Main text must be in English** (the cover page is bilingual/Chinese; keep it as-is).
2. **Do not fabricate citations or quotes.**
   - If a claim needs a source and you don’t have one, leave a `TODO` note or ask for a source.
3. **Cite every factual claim** (dates, attributions, “firsts”, definitions, historical narratives, numerical values).
4. Course requirements (summary): 50 pages total (cover excluded), page numbers, PDF, and at least 5 references with a healthy mix of primary sources and reliable secondary research.

## Research workflow expectations

When drafting any `\subsubsection{...}` that currently contains “To Finish”:

- Gather **at least 2 independent sources** for key facts.
- Prefer:
  - primary sources (original texts / translations / original papers), and
  - scholarly secondary sources (books, peer‑reviewed articles).
- Use `大一/正文/MathHistoryPaper/research/claim-ledger.md` to track:
  - the claim,
  - the source(s),
  - and the BibTeX key(s) you used.

## Final networking/search policy (synchronized)

For this repository, use the following default online research route:

1. Use `#websearch` / `@websearch` in VS Code to discover candidate sources.
   - In the current VS Code extension path, this route is Tavily-backed and requires a Tavily API key.
2. Use `fetch` MCP to retrieve full content from selected URLs and recursively collect relevant linked pages.
3. Use `openalex-database` to verify scholarly metadata (DOI, author/work metadata, citation context).
4. Use `citation-management` to normalize and validate BibTeX entries before adding them to `references.bib`.

Operational constraints:

- Do not cite search snippets. Cite original sources only.
- If a factual claim still lacks a verifiable source, leave a `TODO` instead of guessing.
- GitHub account-level Copilot web search policy may reference Bing. Treat that as platform policy context, not this repository's default operational route unless explicitly switched.
- Use `web-access` only as a fallback when static retrieval fails (login-gated pages, dynamic rendering, required interaction).
- For this project, `web-access` is sufficient for practical browser interaction needs; dedicated Playwright MCP tooling is optional and only needed for broader automated E2E/testing workflows.

## LaTeX conventions

- Preserve the existing structure and formatting choices (margin, spacing, ctex cover page, etc.).
- Use `cleveref` (`\cref{...}`) for cross-references.
- Use `\textit{}` for book titles.
- Keep diagrams vector-based when possible (TikZ/PGFPlots). For external images, ensure licensing is compatible and cite the source.

## Citations & bibliography

- Add sources to `references.bib` and cite them with `\cite{Key}`.
- Use page pinpoint citations when possible: `\cite[pp.~123--125]{Key}`.
- For websites, use `@online` with `url` and `urldate`.

## How to validate changes

If you are asked to verify the document builds, compile with XeLaTeX + biber (LaTeX Workshop recipe is fine). A typical sequence is:

- XeLaTeX → Biber → XeLaTeX → XeLaTeX

Do not change the build toolchain unless necessary.

---
> Source: [SoFarSoGoodya/math-history-paper](https://github.com/SoFarSoGoodya/math-history-paper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-04 -->
