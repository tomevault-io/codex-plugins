# 📌 Project Overview
This repository is a **small React application scaffolded with Vite**. It renders a static list of guitars and emulates a shopping cart. No backend or API exists; state is kept in memory.

### Key pieces
* `src/App.jsx` – root component. Loads static data and holds top‑level state (`data`, `cart`).
* `src/components/` – presentational pieces:
  * `Guitar.jsx` – individual guitar card, receives a `guitar` object and `setCart` (cart state is passed but not always used).
  * `Header.jsx` – page header with bootstrap markup; includes a hard‑coded cart UI overlay.
* `src/data/db.js` – array of guitar objects used by `App.jsx`. To add sample items, drop a new object here.
* `public/img/` – static images referred by components via paths like `img/guitarra_01.jpg`.
* `index.html` – Vite entry template.

The application uses **functional components with hooks** (`useState`). Files are `.jsx` but the project is configured as an ES module (`"type": "module"` in `package.json`).

### Build / Dev workflow
The following npm scripts are the standard developer commands:
```bash
npm install           # install dependencies
npm run dev           # start Vite dev server (HMR enabled)
npm run build         # production build into /dist
npm run preview       # locally serve built output
npm run lint          # run ESLint over `src` and the workspace
```
There are no tests; linting is the only automated check.

### ESLint configuration
Rules live in `eslint.config.js`. React Hooks and Vite refresh plugins are enabled. The only custom rule is:
```js
'no-unused-vars': ['error', { varsIgnorePattern: '^[A-Z_]' }]
```
Use `npm run lint` or configure your editor to pick up `.eslintrc` equivalence.

### Patterns & conventions
* All components live under `src/components` and export a default function.
* Props are passed explicitly and destructured at the top of each component.
* Styling is done with Bootstrap CSS classes defined in `index.css`.
* Image paths reference the `public` folder directly; e.g. `src="img/${image}.jpg"` for guitars and `src="./public/img/logo.svg"` in the header.
* Data from `db.js` is immutable; to simulate dynamic data you'd copy it into state in `App.jsx`.
* Cart state is managed in `App.jsx`; to display or modify it you must lift state further up and pass both `cart` and `setCart` as props.
* There is no routing or context API used.

### Tips for AI agents
* When adding a new feature, start at `App.jsx` to see where state lives and what is rendered.
* To update visuals or layout, inspect the Bootstrap class names used in components.
* Images must be placed in `public/img` and referenced exactly as in existing code.
* Use the same import style (`import X from "./path"`) to stay consistent.
* Lint fixes can be performed automatically by running `npm run lint -- --fix`.
* The `vite.config.js` is mostly default; it includes the React SWC plugin. Adjust only if you need new resolve aliases.

> **Note:** there are a couple of small inconsistencies in the code (e.g., `Guitar` component doesn't destructure the passed-in `cart` prop). Treat these as hints when extending functionality.

Please review and let me know if any crucial pattern or workflow is missing or unclear!

---
> Source: [jorge-DPS/Curso-React](https://github.com/jorge-DPS/Curso-React) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-13 -->
