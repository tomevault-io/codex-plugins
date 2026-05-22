# Copilot Instructions for AI Coding Agents

## Project Overview
- **Framework:** Next.js (TypeScript)
- **Containerization:** Docker & Docker Compose for all development and deployment workflows
- **App Structure:**
  - `app/`: Next.js app entry, global styles, layout
  - `src/components/`: Atomic design (atoms, molecules, organisms, templates)
  - `src/contexts/`: React context providers (e.g., MouseCursorContext)
  - `src/hooks/`: Custom React hooks
  - `public/`: Static assets (SVGs, icons)

## Key Workflows
- **Development:**
  - Always run locally via Docker Compose (`docker compose up`)
  - Hot reload is enabled; if unreliable, set `CHOKIDAR_USEPOLLING: "true"` and `WATCHPACK_POLLING: "true"` in `docker-compose.yml`
  - Do not use Windows file system for project files (use WSL2/Linux/Mac)
- **Dependency Management:**
  - Install npm packages inside the running container (`docker compose exec nextjs-app sh` → `npm install <package>`)
  - Rebuild container after adding/removing packages to sync `node_modules`
- **Testing:**
  - Uses Jest (`jest.config.js`, `jest.setup.js`)
  - Test files are colocated (e.g., `Component.test.tsx`)
  - Run tests inside the container for environment consistency
- **Troubleshooting:**
  - To reset: `docker compose down -v` and `docker volume prune`, then rebuild
- **Component Generation (Atomic Design):**
  - Use custom scripts powered by `generate-react-cli` to scaffold new components:
    - Atom: `npm run atom` (or `npm run gen:atom`)
    - Molecule: `npm run molecule` (or `npm run gen:molecule`)
    - Organism: `npm run organism` (or `npm run gen:organism`)
    - Template: `npm run template` (or `npm run gen:template`)
  - These scripts create components in the correct atomic design subfolder under `src/components/`

## Patterns & Conventions
- **Atomic Design:**
  - Components are organized as atoms, molecules, organisms, templates
  - Example: `src/components/atoms/CursorFactory/CursorFactory.tsx`
- **Context & Hooks:**
  - Contexts in `src/contexts/`, hooks in `src/hooks/`
  - Use React context for cross-component state (e.g., mouse cursor)
- **Styling:**
  - SCSS modules per component (e.g., `Component.scss`)
- **Integration Points:**
  - Next.js routing via `app/`
  - Static assets in `public/`
  - Docker for all builds, runs, and package management
- **Animated Cursor Feature:**
  - The project includes custom cursor logic (see `src/contexts/MouseCursorContext/`, `src/components/atoms/CursorFactory/`, and related components) to provide animated, engaging cursor effects for user interactions and hover states. This is a core part of the site's modern/interactive design direction.

## External Dependencies
- **Next.js**, **React**, **Jest**, **Docker**
- All dependencies must be installed and run inside the Docker container
- **Experimental Packages:**
  - `tone` and `wavesurfer.js` are present for future experiments and can be ignored for most workflows

## Example Commands
- Start app: `docker compose up`
- Build image: `docker compose build`
- Run tests: `docker compose exec nextjs-app npm test`
- Add package: `docker compose exec nextjs-app npm install <package>`

---

For unclear or missing conventions, check `README.md` and the `docker-compose.yml` for up-to-date workflow details.

---
> Source: [vogone1/NextJS-Example-Project](https://github.com/vogone1/NextJS-Example-Project) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
