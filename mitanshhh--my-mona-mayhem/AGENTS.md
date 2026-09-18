# Copilot Instructions for Mona Mayhem

Mona Mayhem is an Astro-based workshop template for building a GitHub contribution battle arena. This file helps Copilot sessions work effectively on the codebase.

## Build & Development Commands

### Local Development
```bash
npm run dev
```
Starts the Astro dev server with hot reload at `http://localhost:3000`.

### Production Build
```bash
npm run build
```
Builds the Astro app for production into the `dist/` directory with server-side rendering via Node adapter.

### Preview Built Output
```bash
npm run preview
```
Serves the production build locally for testing before deployment.

### Direct Astro CLI
```bash
npm run astro -- [command]
```
Access the full Astro CLI (e.g., `astro add`, `astro info`).

## Architecture

### Framework & Runtime
- **Astro v6** with TypeScript strict mode (`astro/tsconfigs/strict`)
- **Node.js adapter** in standalone mode for server-side rendering
- Output mode: `server` (dynamic rendering)
- File-based routing: `src/pages/` routes become URL endpoints

### Directory Structure
- **`src/pages/`** — Astro pages and API routes via file-based routing
  - `src/pages/index.astro` — Main landing page (battle page with retro arcade theme)
  - `src/pages/api/` — Backend API endpoints (e.g., `/api/contributions/[username]`)
- **`workshop/`** — Structured learning modules (README format) for the 1-hour workshop
- **`docs/`** — Documentation and assets deployed to GitHub Pages
- **`public/`** — Static assets (favicon, images, etc.)

### API Pattern
API endpoints are Astro functions using the `APIRoute` type:
```typescript
export const GET: APIRoute = async ({ params }) => {
  // Handle dynamic route params: [username].ts receives params.username
  return new Response(JSON.stringify(data), {
    status: 200,
    headers: { 'Content-Type': 'application/json' },
  });
};
```

Set `prerender = false` on dynamic API routes to enable server-side rendering.

### External Data Source
The app fetches GitHub contribution graph data from `https://github.com/{username}.contribs` (GraphQL format). Endpoints are marked as TODO for participants to implement.

## Key Conventions

### Astro Component & Route Boundary
Astro pages use the **frontmatter barrier** (`---`) to separate component logic from markup:
```astro
---
// Script section (TypeScript, imports, component logic)
import SomeComponent from '../components/SomeComponent.astro';
---

<!-- Markup section (HTML/Astro template syntax) -->
<SomeComponent />
```

### File-Based Routing
- `src/pages/index.astro` → `/`
- `src/pages/about.astro` → `/about`
- `src/pages/api/contributions/[username].ts` → `/api/contributions/{username}` (dynamic)

### Response Handling
All API responses include explicit `Content-Type` headers and appropriate HTTP status codes (e.g., `501 Not Implemented` for TODOs, `200 OK` for success).

### TypeScript Imports
Use ESM syntax (`import`/`export`) and explicit type annotations from `astro` for API functions.

## Design Guide: Retro Arcade Theme

### Color Palette (Neon Aesthetic)
- **Background:** `#0a0a1a` (Deep dark blue/black)
- **Primary Accent (Neon Green):** `#5fed83` with glow effects
- **Secondary Accent (Neon Purple):** `#8a2be2` for gradients and pulses
- **Error Color:** `#ff006e` with neon glow
- **Text Primary:** `#5fed83` (matches accent green)
- **Text Secondary:** `#8a2be2` (subtle purple text)
- **Glows:** Use `text-shadow`, `box-shadow` with 10-30px blur and semi-transparent colors

### Typography
- **Font:** Press Start 2P (loaded from Google Fonts)
- **Fallback:** `cursive` for system fonts
- **Apply to:** `h1`, `h2`, buttons, labels, all interactive elements
- **Letter Spacing:** 2-4px on titles, 1-2px on body text
- **Font Size:** Reduce by ~20% on mobile (media query `max-width: 768px`)

### Animation Patterns
- **Neon Glow Pulse:** `neon-glow` keyframes (2s ease-in-out) on titles
- **Electrical Flicker:** `electrical-flicker` (0.8s) for subtle on/off effect on title text
- **CRT Scanlines:** `crt-flicker` animation on `body::before` overlay (repeating-linear-gradient)
- **Gradient Rotation:** `gradient-rotate` (4s) for VS badge and buttons
- **Shimmer Sweep:** `shimmer-sweep` on `.player-card::after` (diagonal light sweep)
- **Float-in:** `float-in` (0.6s staggered delays) on inputs and labels
- **Color Shift:** `color-shift` (2s alternating green/purple) on loading text

### Border & Shadow Styling
- **Borders:** All use 3px solid neon green (`#5fed83`), no border-radius (arcade style: sharp corners)
- **Box Shadows:** Double-layer glow patterns:
  - Hover: `0 0 30px rgba(95, 237, 131, 0.8), 0 0 50px rgba(138, 43, 226, 0.4)`
  - Default: `0 0 10-20px rgba(95, 237, 131, 0.3)`
  - Inset for cards: `inset 0 0 20px rgba(95, 237, 131, 0.1)`
- **Text Shadows:** `0 0 8-10px color` to match text color

### Interactive Elements
- **Buttons & Inputs:**
  - Border: 3px solid neon green (`#5fed83`), no radius
  - Background: `#0a0a1a` by default
  - Hover: Swap (background becomes neon green, text becomes dark)
  - Scale transform on hover: `scale(1.05)`
  - Transition: 0.2s all (smooth but snappy)
- **Contribution Grid Squares:**
  - Hover: Scale 1.15, dual neon green glow
  - Cursor: crosshair
  - Colors: GitHub palette (gray to dark green per contribution level)

### Pseudo-Elements & Overlays
- **`body::before` (CRT Scanlines):** Fixed overlay, repeating-linear-gradient (2px white bars), opacity 0.03, pointer-events none
- **`.player-card::after` (Shimmer):** Diagonal linear-gradient sweep left-to-right, animation 2s, every 5s

### Responsive Behavior (Mobile, max-width: 768px)
- Font sizes reduce by ~20%
- Button padding shrinks to `0.6rem 1.5rem`
- Grid layout collapses to single column
- VS badge hidden
- CRT effect may be disabled for performance

### Accessibility
- **Motion Preferences:** Support `prefers-reduced-motion` media query (disable animations for users who request it)
- **Contrast:** Neon green on dark background meets WCAG AA standards
- **Focus States:** Ensure inputs have clear focus glow (border-color + box-shadow change)

### Rule for New UI
**All new UI components must maintain the neon arcade aesthetic:**
- Use the color palette (green, purple, dark background)
- Apply Press Start 2P font to interactive elements
- Include glow effects on hover/focus
- No rounded corners (arcade cabinets have sharp edges)
- Stagger animations for float-in effects on new sections
- Apply neon borders (3px solid green)

## Deployment Notes

The default `.github/workflows/deploy.yml` deploys static docs to GitHub Pages (workshop materials in `docs/` and `workshop/`), **not** the Astro app.

To deploy the Astro app to production:
- If using **GitHub Pages** (static): switch `output` to `static` and remove the Node adapter
- If keeping **server-side rendering**: deploy to a Node-compatible host (Vercel, Netlify, Railway, etc.)

See `README.md` for full deployment options.

---
> Source: [mitanshhh/my-mona-mayhem](https://github.com/mitanshhh/my-mona-mayhem) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-18 -->
