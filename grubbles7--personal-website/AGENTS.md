# Copilot Instructions for Personal-Website

## Project Overview
A minimal, performant personal portfolio website hosted on GitHub Pages. Uses vanilla HTML/CSS/JS with a minimalist ASCII aesthetic and system-aware dark mode. No build tools, frameworks, or external dependencies required.

## Key Architecture Patterns

### CSS Variable Design System
All colors and fonts are defined at the root level in `:root` with system dark mode detection (`@media (prefers-color-scheme: dark)`). 
- Light mode: `--bg: #f4f6f8`, `--accent: #007acc`
- Dark mode: `--bg: #000000`, `--accent: #00ffd5`
- When styling new elements, always reference CSS variables, never hardcode colors

### HTML Structure Conventions
- Single-page app with multi-page fallback structure (`index.html`, `about.html`, `portfolio.html` linked in nav)
- `.container` class for max-width layout (1100px) with centered margins
- Semantic HTML: `<header>`, `<nav>`, `<main>`, `<footer>` structure
- Navigation links follow ASCII bracket pattern: `[ Link ]` (rendered via CSS `::before` and `::after` pseudo-elements)

### JavaScript Patterns
- Vanilla JS only; "use strict" mode
- Event delegation via `DOMContentLoaded` for DOM-ready operations
- Intersection Observer API for scroll-based animations (see `.ascii-box` handling in `script.js`)
- CSS transitions applied via `style` property for animation state management
- Selectors use semantic class names (`.ascii-box`, `.container`, `.nav-links`, etc.)

### ASCII Visual Elements
- `.ascii-box` class wraps pre-formatted ASCII art with monospace font
- Applied with glow animation (`@keyframes glow`) tied to color scheme
- Initialized with opacity 0 and translateY(20px) by JS, animated in on intersection
- Use `white-space: pre` to preserve formatting

## File Responsibilities
- **index.html**: DOM structure, semantic HTML, nav system
- **style.css**: All styling; CSS variables at top, grouped by section (Reset → Layout → Header → Content → Footer → Animations)
- **script.js**: DOM interactions, Intersection Observer setup for `.ascii-box` elements

## Development Workflow
- No build tools: edit files directly and test in browser
- CSS changes apply immediately; refresh browser to see updates
- JS changes require page reload (browser cache may interfere; use Dev Tools Cmd+Shift+J)
- Dark mode testing: use browser DevTools to toggle `prefers-color-scheme` emulation

## Common Tasks
**Add a new page**: Create `newpage.html` with same header/footer structure, add nav link in header
**Add ASCII art**: Wrap in `<pre class="ascii-box">` element; glow animation applies automatically
**Update colors**: Modify CSS variables in `:root` and `@media (prefers-color-scheme: dark)` block
**Add interactivity**: Add event listener in `script.js` within `DOMContentLoaded` callback

---
> Source: [Grubbles7/Personal-Website](https://github.com/Grubbles7/Personal-Website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-12 -->
