## Copilot guidance — Gigzi landing site

This project is a small React + Vite landing site for Gigzi. The guidance below highlights code patterns, integration points, and commands that help an AI agent make safe, correct changes.

### Quickstart & developer workflow
- Run the dev server: `npm install` then `npm run dev` (Vite + HMR). Default dev URL: http://localhost:5173
- Build for production: `npm run build` then `npm run preview` to locally preview the optimized build.
- Lint: `npm run lint` uses ESLint configuration included in the repo.

Notes about environment variables
- Vite uses `import.meta.env`. Example used here: `import.meta.env.VITE_RAZORPAY_KEY` (see `src/components/Pay.jsx`).
- Add per-machine variables to a `.env` or `.env.local` file at the repository root. Example file contents:

  VITE_RAZORPAY_KEY=your_razorpay_key_here

  After editing `.env` files restart the dev server.

### Architecture & important files
- `index.html` — app entry; look at `root` element used by `createRoot` in `src/main.jsx`.
- `src/main.jsx` — app bootstrap, imports `index.css`.
- `src/App.jsx` — client-side routing using `react-router-dom`. The base routes and page components live here; changes to navigation and route structure should be made here.
- `src/components/*` — page-level routes and UI components. Look for these pages: `Landing`, `Register` (artist signup), `Pay`, `Success`, `Profile`, and `InternshipPage`.

### Patterns & integrations (be explicit)
- Tailwind CSS: imported in `src/index.css` (`@import "tailwindcss"`). Styles are implemented with Tailwind utility classes across components.
- API calls: many components use `axios` directly (`src/components/Register.jsx`, `src/components/Pay.jsx`, `src/components/Success.jsx`). The `Register` page uses a `BASE_URL` constant and contains helper `getMethod`/`postMethod` functions — keep URL patterns consistent when adding endpoints.
- Cloudinary uploads: `Register.jsx` uploads files to Cloudinary using a preset `ARTIST_DOCS` and account `dkonh16z8`. If altering upload behavior, update the preset accordingly.
- Payment flow: Razorpay is used in `Pay.jsx` and `Success.jsx`. `Pay.jsx` requires `import.meta.env.VITE_RAZORPAY_KEY` and constructs Razorpay options. `Success.jsx` verifies payment using a backend endpoint. When working on payments, keep existing `window.ReactNativeWebView?.postMessage(...)` calls intact — they are used to communicate status back to a mobile WebView host.

### Conventions the AI should follow
- Files in `src/components/` are React function components; use PascalCase component names and default exports.
- UI-driven logic (forms, uploads) live inside page components — prefer minimal refactors that preserve network helper patterns already in the component unless you centralize API calls consistently across the codebase.
- Keep Tailwind utility classes in JSX; avoid introducing global CSS unless necessary. `src/App.css` contains local layout rules.

### Debugging & local testing hints
- Browser console and React DevTools will show runtime errors. Vite logs helpful compile-time errors in the terminal.
- To inspect payment flow locally: hit `/pay?orderId=...` with a valid backend order id (requires backend service). Razorpay keys must be set via `.env`.
- Mobile integration: `window.ReactNativeWebView.postMessage(...)` is present in `Pay.jsx` and `Success.jsx` — use those logs as a communication contract when editing payment or order code.

### External services to be aware of
- Cloudinary (image uploads): preset `ARTIST_DOCS`, `dkonh16z8` account.
- Backend services: examples hard-coded inside components:
  - `https://gigzi-gigzi.vercel.app` — used in `Register.jsx`
  - `https://gigzi-dev.vercel.app` — used in `Pay.jsx`
  - `https://gigzii-backend-e41i.vercel.app` — used in `Success.jsx`

When editing API paths, search repo for `gigzi` base URL usage and update all related endpoints as necessary.

### Security & privacy note
- The `Register` form uploads Aadhaar images (sensitive). Be careful with any changes that persist or log these uploads. Keep Cloudinary upload preset and server-side verification unchanged unless you control the backend.

### Example tasks & where to implement them
- Add a new route/page: update `src/App.jsx` and add component to `src/components`.
- Change API behavior for signup: update helper functions in `src/components/Register.jsx` or centralize them to a new `src/api/*` module and refactor callers.
- Update payment flow: edit `src/components/Pay.jsx` (init) and `src/components/Success.jsx` (verify) — maintain `window.ReactNativeWebView` hooks.

If anything is unclear or you want the agent to also add tests or create an API abstraction, tell me which direction and I’ll iterate on this file.

---
> Source: [Gigzii4/gigzi_landing](https://github.com/Gigzii4/gigzi_landing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-13 -->
