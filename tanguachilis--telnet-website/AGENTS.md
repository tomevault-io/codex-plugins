# Copilot Instructions — Telnet Cameroon Website

## Project Overview

Marketing/portfolio website for **Telnet Cameroon**, an IT solutions company in Buea, Cameroon. Built with **React 19 + Vite 7**, using **react-router-dom v7** for client-side routing. Pure CSS styling (no UI library). No backend — all "contact" actions redirect to WhatsApp.

## Architecture

### Component-Page Pattern

Every section (Services, About, Team, Gallery, Shop, Contact) exists as a **reusable component** in `src/components/` and a **thin page wrapper** in `src/pages/`. Pages add a `PageBanner` header and render the component with `showHeader={false}`:

```jsx
// src/pages/ServicesPage.jsx — typical page pattern
<div className="page-banner">...</div>
<Services showHeader={false} />
```

Components accept `showHeader = true` by default so they render standalone on the HomePage.

### Routing

All routes defined in `src/App.jsx`. Seven flat routes: `/`, `/services`, `/about`, `/team`, `/gallery`, `/shop`, `/contact`. Layout is `Navbar` + `<Routes>` + `Footer` + `WhatsAppFloat`, wrapped in `<BrowserRouter>`.

`ScrollToTop` resets scroll on navigation. `AnimationObserver` uses `IntersectionObserver` to add `.visible` class to `.animate-on-scroll` elements on route change.

### WhatsApp Integration (no backend)

All CTAs route to WhatsApp via `src/utils/whatsapp.js`. This is the primary contact mechanism — there is no form submission endpoint.

- `getWhatsAppUrl(message)` — builds `wa.me` URL with pre-drafted text
- `whatsappMessages` — object of keyed message templates (e.g., `laptopSales`, `cctv`, `quote`)
- Components reference messages by key: `whatsappMessages[cat.whatsappKey]`
- Phone number constant: `237671827893`

When adding new services or products, add a corresponding message key to `whatsappMessages` in `src/utils/whatsapp.js`.

## Key Conventions

### Styling

- **One CSS file per component**: `ComponentName.css` imported directly in `ComponentName.jsx`
- Global styles in `src/index.css` and `src/App.css`
- Shared page banner styles in `src/pages/PageBanner.css`
- CSS class conventions: `glass-card` for frosted-glass card style, `.animate-on-scroll` for scroll-triggered animations, `.section-header` / `.section-label` / `.section-title` / `.section-subtitle` for consistent heading blocks
- Fonts: Inter (body) + Outfit (headings), loaded via Google Fonts in `index.html`

### Data Patterns

Content data (services, team members, shop categories, gallery images) is defined as **const arrays at the top of each component file**, not in separate data files. To add/edit content, modify these arrays directly.

### Static Assets

- Team photos and images: `public/Our team/` (spaces in path — always quote in URLs)
- Image references use absolute paths from public root: `"/Our team/logo.png"`, `"/Our team/other images/training sesseions.jpg"`
- Logo: `/Our team/logo.png`

### Component Exports

All components use `export default function ComponentName()` syntax (named function default exports).

### Icons

Inline SVGs throughout — no icon library. Copy existing SVG patterns when adding new icons.

## Development

```bash
npm run dev      # Start Vite dev server with HMR
npm run build    # Production build
npm run preview  # Preview production build
npm run lint     # ESLint
```

## Adding a New Page

1. Create component in `src/components/NewSection.jsx` + `NewSection.css`
2. Create page in `src/pages/NewSectionPage.jsx` using the `PageBanner` + `showHeader={false}` pattern
3. Add route in `src/App.jsx`
4. Add nav link to `navLinks` array in `src/components/Navbar.jsx`
5. Add footer link in `src/components/Footer.jsx`
6. If it has a WhatsApp CTA, add message key to `src/utils/whatsapp.js`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/TanguAchilis) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
