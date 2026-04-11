<!-- markdownlint-disable -->

# Gemini Project Configuration

## Project Overview

This is a simple "todos" application with a Go backend and a PostgreSQL database. The entire application is containerized using Docker.

## Technology Stack

- **Backend:** Go
- **Database:** PostgreSQL (using the `pgvector/pgvector:pg17` image)
- **Containerization:** Docker, Docker Compose
- **Frontend:** React, Vite, TypeScript, Tailwind CSS, Shadcn/ui, TanStack Router, TanStack Query, Tauri

## Key Commands

- **Run the application:** `docker-compose up --build`
- **Stop the application:** `docker-compose down`
- **Run the frontend (web):** `bun run dev` (inside the `clients` directory)
- **Run the frontend (desktop):** `bun run tauri dev` (inside the `clients` directory)
- **Add shadcn/ui component:** `bunx --bun shadcn@latest add {component}` (inside the `clients` directory)

## Database Connection

- **Service Name (within Docker):** `postgres`
- **Local Connection URL:** `postgresql://postgres:postgres@localhost:5432/postgres`
- **Configuration:** The Go application reads database credentials from the `server/.env` file.

## Frontend Structure

```
clients/
├── bun.lock
├── components.json
├── index.html
├── node_modules/
├── package.json
├── public/
├── README.md
├── src/
│   ├── App.css
│   ├── App.tsx
│   ├── assets/
│   ├── components/
│   │   └── Chat.tsx
│   ├── lib/
│   ├── main.tsx
│   ├── routes/
│   │   ├── index.lazy.tsx
│   │   └── routeTree.gen.ts
│   ├── vite-env.d.ts
├── src-tauri/
├── tsconfig.json
├── tsconfig.node.json
└── vite.config.ts
```

**Key Observations:**

- **`bun.lock`**: Bun's lock file for dependency management.
- **`components.json`**: Configuration file for `shadcn/ui`, defining aliases and other settings.
- **`index.html`**: The main HTML entry point for the web application.
- **`node_modules/`**: Directory containing installed Node.js modules.
- **`package.json`**: Project metadata and script definitions.
- **`public/`**: Static assets directory.
- **`README.md`**: Project README file.
- **`src/`**: Source code directory.
  - **`App.css`**: Global CSS for the application.
  - **`App.tsx`**: The root React component, responsible for setting up TanStack Router.
  - **`assets/`**: Directory for static assets like images.
  - **`components/`**: Contains reusable React components.
    - **`Chat.tsx`**: The main chat UI component, using `shadcn/ui` components.
  - **`lib/`**: Likely contains utility functions or helper modules.
  - **`main.tsx`**: The entry point for the React application, rendering `App.tsx` and setting up `QueryClientProvider`.
  - **`routes/`**: Contains route definitions for TanStack Router.
    - **`index.lazy.tsx`**: Lazy-loaded route for the home page, rendering the `Chat` component.
    - **`routeTree.gen.ts`**: Automatically generated file by TanStack Router, defining the route tree.
  - **`vite-env.d.ts`**: TypeScript declaration file for Vite environment variables.
- **`src-tauri/`**: Contains the Tauri-specific source code and configuration for the desktop application.
- **`tsconfig.json`**: TypeScript configuration for the client-side code, including path aliases.
- **`tsconfig.node.json`**: TypeScript configuration for Node.js specific files (e.g., Vite config).
- **`vite.config.ts`**: Vite build configuration, including plugins for React and TanStack Router.

This structure indicates a well-organized React application using Vite, Bun, Tailwind CSS, Shadcn UI, TanStack Router, and TanStack Query, with support for both web and desktop (Tauri) environments.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sereschen)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/sereschen)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
