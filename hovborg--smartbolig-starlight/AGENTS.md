# SmartBolig.net - Copilot Instructions

## Projekt Kontekst

Dette er **smartbolig.net** - en dansk smart home tutorial website bygget med:
- **Astro Starlight** (dokumentations-framework)
- **Tailwind CSS** (styling)
- **IBRACORP-inspireret dark theme** (GitHub Dark farver)
- **Cloudflare Pages** (hosting)

## Tech Stack

- Framework: Astro 5.x + Starlight 0.37.x
- Styling: Tailwind CSS 4.x + Custom CSS
- Languages: Danish (da) default, English (en)
- Build: Static site generation
- Deploy: Cloudflare Pages via Wrangler

## Farve Palette (GitHub Dark / IBRACORP)

```css
--bg-primary: #0d1117
--bg-secondary: #161b22
--bg-tertiary: #1a2332
--text-primary: #e6edf3
--text-secondary: #8b949e
--accent-blue: #58a6ff
--accent-green: #3fb950
--accent-green-dark: #238636
--border: #30363d
```

## Fil Struktur

```
src/
├── content/docs/
│   ├── da/              # Dansk indhold
│   │   ├── index.mdx    # Forside
│   │   ├── home-assistant/
│   │   ├── esp32/
│   │   ├── automationer/
│   │   ├── produkter/
│   │   └── sikkerhed/
│   └── en/              # Engelsk indhold
├── styles/
│   ├── global.css       # Tailwind import
│   └── custom.css       # IBRACORP theme
└── assets/
    ├── logo-dark.svg
    └── logo-light.svg
```

## Når du hjælper med kode

1. **Brug danske kommentarer** i config filer
2. **Følg Starlight conventions** for frontmatter
3. **Brug CSS custom properties** fra custom.css
4. **Test med `npm run dev`** før deploy

## Frontmatter Format

```mdx
---
title: "Sidetitel"
description: "Beskrivelse til SEO"
sidebar:
  order: 1
---
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Hovborg)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Hovborg)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
