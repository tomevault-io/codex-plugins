# Copilot Instructions

## Build Commands

```bash
npm install        # Install dependencies
npm run dev        # Start dev server on an available port
npm run build      # Production build
```

There are no tests or linters configured.

## Architecture

This is a React 19 marketing site for GPUcloud.xyz using `react-router-dom`. `App.tsx` keeps a `ViewState` union and maps it to URL routes (for nav state + component compatibility). Components still receive `onNavigate: (view: ViewState) => void` and `App.tsx` converts that to route navigation.

All page and UI components live in a flat `components/` directory. Each page component corresponds to a value in the `ViewState` type. When adding a new page:

1. Create the component in `components/`
2. Add its key to the `ViewState` union type in `App.tsx`
3. Add `viewToPath`/`pathToView` mapping entries in `App.tsx`
4. Add a `<Route />` entry in the shared route list in `App.tsx`

Two pages (`dashboard`, `console`) render without the shared `Header`/`Footer` and use their own full-page layouts with early returns in `App.tsx`.

## Styling

Tailwind CSS is loaded via **CDN script tag** in `index.html`—there is no PostCSS or Tailwind config file. The Tailwind theme is extended inline in `index.html` with a custom `dats` color palette:

- `dats-dark`: `#000000` (true black)
- `dats-blue`: `#3b82f6` (primary accent)
- `dats-light`: `#0a0a0a`
- `dats-gray`: `#222222`
- `dats-border`: `#ffffff`

Custom utility classes are defined via `<style>` tags in `App.tsx` (and duplicated in special-layout pages):

- `sharp-shadow` / `sharp-shadow-sm` / `sharp-shadow-blue` — box shadows for the brutalist high-contrast aesthetic
- `bg-grid-pattern` — grid background overlay

Fonts: **JetBrains Mono** (default `font-sans` and `font-mono`) and **Space Grotesk** (`font-display`), loaded via Bunny Fonts CDN in `index.html`.

## Conventions

- Components use `React.FC` with explicit prop interfaces
- Icons from `lucide-react` exclusively
- Path alias `@/*` maps to the project root (configured in `tsconfig.json` and `vite.config.ts`)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/todobig)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/todobig)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
