# Copilot Instructions for InnoHealth Africa Technology Web App

## Project Overview
- **Stack:** React (TypeScript), Vite, Tailwind CSS, Framer Motion
- **Purpose:** Modern, responsive web app for InnoHealth Africa Technology
- **Key Features:** Animated cards, overlays, accessible UI, clean component structure

## Architecture & Patterns
- **Entry Point:** `src/main.tsx` mounts `App.tsx` to the DOM
- **Component Structure:**
  - `src/components/sections/` — Major page sections (e.g., `HeroSection.tsx`, `Navbar.tsx`)
  - `src/components/ui/` — Reusable UI primitives (e.g., `button.tsx`, `dropdown-menu.tsx`)
- **Styling:** Tailwind CSS via `index.css` and `brand.css`; config in `tailwind.config.js`
- **Animation:** Framer Motion is used for interactive/animated UI elements
- **No Redux or Context API:** State is managed locally within components

## Developer Workflows
- **Install:** `npm install`
- **Dev Server:** `npm run dev` (Vite, hot reload)
- **Build:** `npm run build` (output in `build/`)
- **No formal test suite** (add tests in `src/` if needed)
- **Deploy:** Netlify config in `netlify.toml` and `_redirects` files

## Conventions & Practices
- **Component Naming:** PascalCase for files and exports
- **Section Components:** Each major section in `sections/` is a self-contained React component
- **UI Primitives:** Use `ui/` components for buttons, dropdowns, and image fallbacks
- **CSS:** Prefer Tailwind utility classes; use `brand.css` for custom styles
- **No Figma/Windows artifacts:** Codebase is cleaned for production

## Integration & External Dependencies
- **Framer Motion:** For animation (see usage in section components)
- **Netlify:** Deployment via `netlify.toml` and `_redirects`
- **Vite:** Fast dev/build tooling (`vite.config.ts`)

## Examples
- To add a new section: create a file in `src/components/sections/`, export a React component, and import it in `App.tsx`
- To add a new button style: extend `src/components/ui/button.tsx` and reference in section components

## References
- Main app: `src/App.tsx`
- Section pattern: `src/components/sections/HeroSection.tsx`
- UI primitive: `src/components/ui/button.tsx`
- Styling: `src/index.css`, `src/brand.css`, `tailwind.config.js`
- Build config: `vite.config.ts`

---
For questions, see `README.md` or contact the InnoHealth Africa Technology team.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mudimurtala) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
