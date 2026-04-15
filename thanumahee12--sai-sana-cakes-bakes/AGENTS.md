# Project: Sai Sana Cakes & Bakes Website

This project is a modern web application for a cake and bakeshop, built using a React starter template. The initial setup provides a robust foundation for developing a rich, interactive user experience.

## Project Overview

*   **Purpose:** To build a visually appealing, performant e-commerce and portfolio website for a cake business.
*   **Core Technologies:**
    *   **Frontend Framework:** React 18
    *   **Build Tool:** Vite
    *   **Routing:** React Router DOM
    *   **Linting & Formatting:** ESLint and Prettier
*   **Planned Technologies (To be implemented):**
    *   **3D Graphics:** Three.js with React Three Fiber
    *   **Styling:** Tailwind CSS
    *   **Animations:** Framer Motion
    *   **Backend Services:** Firebase (Hosting, Firestore, Auth) and Supabase (Storage)

## Building and Running

The project uses `npm` as the package manager.

*   **Install Dependencies:**
    ```bash
    npm install
    ```

*   **Run Development Server:**
    Starts the server on `http://localhost:3000`.
    ```bash
    npm run dev
    ```

*   **Create Production Build:**
    Bundles the application into the `dist/` directory.
    ```bash
    npm run build
    ```

*   **Preview Production Build:**
    Serves the `dist/` directory locally to test the production build.
    ```bash
    npm run preview
    ```

*   **Lint Code:**
    Analyzes the code for style and syntax errors using ESLint.
    ```bash
    npm run lint
    ```

*   **Format Code:**
    Automatically formats code using Prettier.
    ```bash
    npm run format
    ```

## Development Conventions

*   **Folder Structure:** The `src` directory is organized by feature/responsibility:
    *   `src/assets`: Static assets like images and fonts.
    *   `src/components`: Reusable React components.
    *   `src/constants`: Application-wide constants.
    *   `src/contexts`: React Context providers for global state.
    *   `src/hooks`: Custom React hooks.
    *   `src/layout`: Layout components (e.g., `MainLayout` with header/footer).
    *   `src/reducers`: State management logic.
    *   `src/routes`: Route definitions using `react-router-dom`.
    *   `src/styles`: Global CSS files (`reset.css`, `global.css`).
    *   `src/utils`: Utility functions.

*   **Path Aliases:** Path aliases are configured in `vite.config.js` and `jsconfig.json` for cleaner import statements:
    *   `@/` → `src/`
    *   `@components/` → `src/components/`
    *   `@utils/` → `src/utils/`
    *   `@contexts/` → `src/contexts/`
    *   `@routes/` → `src/routes/`
    *   `@layout/` → `src/layout/`
    *   `@styles/` → `src/styles/`
    *   `@assets/` → `src/assets/`
    *   `@constants/` → `src/constants/`
    *   `@reducers/` → `src/reducers/`
    *   `@hooks/` → `src/hooks/`

*   **Routing:** The application uses `react-router-dom` with a file-based routing convention defined in `src/routes/index.jsx`. A main layout component (`src/layout/MainLayout.jsx`) wraps the pages, providing a consistent structure. The router needs to be integrated into `src/main.jsx` to be active.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ThanuMahee12)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/ThanuMahee12)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
