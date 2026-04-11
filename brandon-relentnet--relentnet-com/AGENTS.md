# Relentnet Project Overview

This project is the website for "Relentnet", a modern marketing/landing page application built with the latest React ecosystem tools. It is designed for high performance, ease of maintenance, and a visually striking dark-themed UI.

## Tech Stack

- **Framework:** [React 19](https://react.dev/)
- **Language:** [TypeScript](https://www.typescriptlang.org/)
- **Build Tool:** [Vite 7](https://vitejs.dev/)
- **Routing:** [TanStack Router](https://tanstack.com/router) (File-based routing)
- **Styling:**
  - [Tailwind CSS 4](https://tailwindcss.com/)
  - [DaisyUI 5](https://daisyui.com/) (Custom "darknet" theme)
  - Custom CSS variables and `@apply` rules in `src/styles.css`
- **Animations:** [Motion](https://motion.dev/) (formerly Framer Motion)
- **Icons:** [Lucide React](https://lucide.dev/), [Heroicons](https://heroicons.com/)
- **Testing:** [Vitest](https://vitest.dev/)
- **Linting/Formatting:** ESLint (TanStack config), Prettier

## Project Structure

```text
/
├── public/                 # Static assets (images, icons, etc.)
├── src/
│   ├── components/         # Reusable UI components
│   │   ├── sections/       # specific page sections (Hero, Footer, etc.)
│   │   └── website/        # Website-specific components
│   ├── data/               # Static data files (e.g., siteData.ts)
│   ├── routes/             # TanStack Router file-based routes
│   │   ├── __root.tsx      # Main layout (Header, Footer, Outlet)
│   │   ├── index.tsx       # Home page
│   │   └── ...             # Other pages
│   ├── main.tsx            # Application entry point
│   ├── styles.css          # Global styles, Tailwind config, DaisyUI theme
│   ├── routeTree.gen.ts    # Auto-generated route definitions
│   └── ...
├── package.json            # Dependencies and scripts
├── vite.config.ts          # Vite configuration
└── tsconfig.json           # TypeScript configuration
```

## Key Commands

| Command           | Description                                      |
| :---------------- | :----------------------------------------------- |
| `npm run dev`     | Start the development server (default port 3000) |
| `npm run build`   | Build the application for production             |
| `npm run preview` | Preview the production build locally             |
| `npm run test`    | Run unit tests with Vitest                       |
| `npm run lint`    | Run ESLint                                       |
| `npm run format`  | Format code with Prettier                        |
| `npm run check`   | Run Prettier check and ESLint fix                |

## Development Conventions

- **Path Aliases:** Use `@/` to import from `src/`.
  - Example: `import { Button } from '@/components/Button'`
- **Routing:** Create new routes by adding files to `src/routes`. TanStack Router automatically generates the route tree.
  - `__root.tsx` acts as the global layout.
- **Styling:**
  - Prefer Tailwind utility classes for layout and spacing.
  - Use the custom DaisyUI theme (`darknet`) colors (e.g., `text-primary`, `bg-base-100`).
  - Complex components can use `@apply` in `src/styles.css` or CSS modules if needed (though `styles.css` seems to be the main place for global component styles).
- **Animations:** Use `motion` components for complex interactions and page transitions.
- **Type Safety:** Strict TypeScript mode is enabled. Ensure all props and state are properly typed.

## Theme Configuration

The project uses a custom DaisyUI theme named **`darknet`** defined in `src/styles.css`.

- **Primary:** `#f38ab8` (Pinkish)
- **Secondary:** `#8e8e8e` (Gray)
- **Accent:** `#56a7db` (Blue)
- **Base:** Dark shades (`#0f0f12`, `#141419`)
- **Font:** Poppins (imported from Google Fonts)

## Data

Static site content seems to be centralized in `src/data/siteData.ts`, making it easy to update text and configurations without modifying components directly.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/brandon-relentnet)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/brandon-relentnet)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
