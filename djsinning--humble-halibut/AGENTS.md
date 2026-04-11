by
a# Copilot Instructions for codespaces-react

## Project Overview
- This is a React single-page application bootstrapped with [Vite](https://vitejs.dev/).
- The entry point is `src/index.jsx`, which renders the main `App` component from `src/App.jsx`.
- Static assets (e.g., images, icons) are located in the `public/` directory and referenced by relative path in components (e.g., `Octocat.png`).

## Key Files and Structure
- `src/App.jsx`: Main UI component. Contains the core layout and example content.
- `src/index.jsx`: Application entry point, responsible for rendering `<App />`.
- `public/`: Static files served at the root. Place images and manifest files here.
- `vite.config.js`: Vite configuration for build and dev server.
- `package.json`: Defines dependencies and scripts.

## Developer Workflows
- **Start Dev Server:** `npm start` (runs Vite dev server, hot reloads on changes)
- **Run Tests:** `npm test` (runs tests in watch mode using Jest or Vitest)
- **Build for Production:** `npm run build` (outputs to `build/`)

## Project Conventions
- Use functional React components and hooks.
- CSS is colocated per component (e.g., `App.css` for `App.jsx`).
- Static assets are referenced with relative paths from `public/`.
- No custom routing or state management libraries are present by default.
- No backend/API integration is included; this is a frontend-only starter.

## Integration Points
- To add new pages or components, create new files in `src/` and import them into `App.jsx`.
- To add static assets, place them in `public/` and reference by filename (e.g., `<img src="Octocat.png" />`).
- For advanced configuration, edit `vite.config.js`.

## Examples
- To display an image from `public/`, use: `<img src="Octocat.png" alt="Octocat" />` in a component.
- To add a new component:
  ```jsx
  // src/MyComponent.jsx
  export default function MyComponent() {
    return <div>Hello!</div>;
  }
  // In App.jsx
  import MyComponent from './MyComponent';
  // ...
  <MyComponent />
  ```

## References
- [Vite Documentation](https://vitejs.dev/guide/)
- [React Documentation](https://reactjs.org/)

---
If you update project structure or add new conventions, please update this file to keep AI agents productive.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DJSINNING)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/DJSINNING)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
