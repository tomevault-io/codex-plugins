# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

A presentation built with [Animotion](https://animotion.pages.dev/) — a Svelte 5 + Reveal.js + Tailwind CSS presentation framework. The presentation is about "Cursor AI — The Art of Talking to Your IDE" for an iO Digital R&D knowledge sharing session.

There are two slide decks: the main one at `/` (slides in `src/slides/`) and a secondary "bonzai" deck at `/bonzai` (slides in `src/bonzai-slides/`).

## Commands

- `pnpm dev` — start dev server
- `pnpm build` — production build
- `pnpm check` — Svelte type checking
- `pnpm lint` — prettier + eslint
- `pnpm format` — auto-format with prettier
- `pnpm test` — run vitest (browser-mode with Playwright/Chromium)

## Architecture

### Slide System

Slides live in numbered directories (e.g., `src/slides/100/slide.svelte`). The number determines sort order — gaps between numbers allow inserting slides later. Slides are auto-discovered via `import.meta.glob` in `src/lib/components/slides.svelte` (and `bonzai-slides.svelte` for the bonzai deck).

Each slide can export `props` via a `<script module>` block using `defineProps()` to configure Reveal.js data attributes (transition, background, auto-animate, etc.). Setting `transition: 'slide'` marks a chapter boundary — consecutive slides without this are grouped into vertical slide sections.

### Core Components (`src/lib/components/`)

- **Presentation** — initializes Reveal.js with plugins (Markdown, Highlight, KaTeX, Notes), manages custom `in`/`out`/`current` events for fragment lifecycle
- **Slide** — wraps a `<section>` with Reveal.js data attributes, listens for `in`/`out` events
- **Slides / BonzaiSlides** — auto-loads and sorts slide modules, groups into chapters
- **Code** — syntax highlighting with Shiki + magic-move animations. Exposes `update`, `selectLines`, `selectToken`, `scrollToLine` tagged template methods
- **Action** — fragment-based step that fires `do`/`undo` callbacks on navigation (forward/backward)
- **Transition** — handles enter/exit animations on fragments

### Key Patterns

- Uses Svelte 5 runes (`$state`, `$effect`, `$props`) throughout
- Animations use `@animotion/motion` tweens
- The `Action` component is the primary way to create step-by-step reveals — bind a `Code` ref and call its methods from Action's `do`/`undo` callbacks
- Tailwind v4 configured via `@tailwindcss/vite` plugin

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/L-ubu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
