# Copilot AI Agent Instructions for ShopSmart Pro Website

## Project Overview
- **Type:** React + Vite web app for ShopSmart Pro browser extension landing/support site
- **Key Directories:**
  - `src/pages/`: Top-level pages (Home, Download, Features, BugReports, etc.)
  - `src/components/`: Shared layout and UI components (many in `ui/`)
  - `src/hooks/`, `src/lib/`: Custom hooks and utility functions
  - `public/`: Static assets (images, favicon)

## Architecture & Patterns
- **Routing:** Each file in `src/pages/` is a page; navigation is handled at the component level (no explicit router file found)
- **UI Components:** Heavy use of composable, atomic UI in `src/components/ui/` (e.g., `button.jsx`, `badge.jsx`, `alert.jsx`). Most are functional components with props for customization.
- **Forms:** Forms (e.g., BugReports) use local React state, not external form libraries.
- **Styling:** Tailwind CSS is used throughout (`index.css`, `input.css`, `tailwind.config.js`). Class composition is common.
- **Icons:** Components often use icon props (e.g., `<AlertTriangle />`, `<Chrome />`)—likely from a shared icon library.

## Developer Workflows
- **Start Dev Server:** `pnpm dev` (or `npm run dev`)
- **Build for Production:** `pnpm build`
- **Preview Build:** `pnpm preview`
- **Lint:** `pnpm lint`
- **Deploy:** Vercel (see `vercel.json`), output is `dist/`

## Conventions & Best Practices
- **Component Naming:** PascalCase for components, camelCase for props/variables.
- **UI Variants:** Many UI components accept a `variant` prop for style changes (see `badge.jsx`, `button.jsx`).
- **Props Forwarding:** Use of `{...props}` and `asChild` pattern for flexible composition.
- **No Redux/Context:** State is local to components; no global state management detected.
- **No API Layer:** No backend/API integration in this repo—site is static/SPA for extension marketing/support.

## Integration Points
- **Extension Download:** `Download.jsx` and `HowItWorks.jsx` describe the Chrome extension install flow.
- **Bug Reporting:** `BugReports.jsx` implements a detailed bug report form (local state, no backend submit).
- **Help/Support:** `HelpCenter.jsx`, `Support.jsx`, and related pages provide static help content.

## Examples
- **UI Variant Example:**
  ```jsx
  <Button variant="outline">Learn More</Button>
  <Badge variant="success">Active</Badge>
  ```
- **Form State Example:**
  ```jsx
  const [formData, setFormData] = useState({ ... })
  <input name="title" value={formData.title} onChange={handleInputChange} />
  ```

## Quick Start
1. Install dependencies: `pnpm install`
2. Start dev server: `pnpm dev`
3. Edit pages/components in `src/`

---
For more, see `README.md` and `package.json` scripts. When in doubt, follow patterns in `src/pages/` and `src/components/ui/`.

---
> Source: [signdrive/ShopSmart-Pro-Site](https://github.com/signdrive/ShopSmart-Pro-Site) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-30 -->
