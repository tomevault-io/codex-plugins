# Copilot Instructions for `bytes-of-art`

This document provides essential guidelines for AI coding agents working on the `bytes-of-art` project. Follow these instructions to ensure consistency, maintainability, and alignment with the project's architecture and conventions.

---

## Project Overview

`bytes-of-art` is an interactive, educational React application that explores art concepts through dynamic visualizations and storytelling. It uses:

- **React Router** for routing
- **TypeScript** for type safety
- **TailwindCSS** for styling
- **Vite** for development and build tooling
- **Docker** for deployment

The app is structured around modular components, with a focus on reusability and responsiveness.

---

## Key Directories

- **`app/`**: Core application logic, including assets, components, data, and routes.
  - `components/`: Reusable UI components (e.g., `ArrowButton`, `ExpandingGallery`).
  - `data/`: Static data files (e.g., `artTimelineData.ts`).
  - `routes/`: Route-specific components and styles.
  - `types/`: Type definitions for data and components.
- **`components/ui/shadcn-io/3d-card/`**: Specialized UI components (e.g., 3D card effects).
- **`pages/`**: Feature-specific pages (e.g., `bento`, `city`, `synesthesia`).
- **`lib/hooks/`**: Custom hooks (e.g., `useGridLayout`, `useMedia`).

---

## Developer Workflows

### Installation

Install dependencies:
```bash
npm install
```

### Development

Start the development server:
```bash
npm run dev
```
Access the app at `http://localhost:5173`.

### Production Build

Create a production build:
```bash
npm run build
```

### Docker Deployment

Build and run the app using Docker:
```bash
docker build -t my-app .
docker run -p 3000:3000 my-app
```

---

## Project-Specific Conventions

1. **Component Naming**:
   - Use descriptive names (e.g., `BentoArtCard` instead of `Card`).
   - Avoid generic names like `data` or `content` for deeply nested objects.

2. **Styling**:
   - Use TailwindCSS utility classes.
   - Extract reusable class combinations using the `cn()` utility in `lib/utils.ts`.

3. **State Management**:
   - Use React Context for global states (e.g., theme, animation settings).
   - Avoid prop drilling by centralizing shared logic.

4. **Data Handling**:
   - Store static data in `app/data/`.
   - Use TypeScript types from `app/types/` to ensure consistency.

5. **Routing**:
   - Define routes in `app/routes.ts`.
   - Use dynamic imports for route-specific components to optimize performance.

---

## Integration Points

- **External Libraries**:
  - TailwindCSS: Styling framework.
  - React Router: Routing.
  - Vite: Build and development tooling.

- **Cross-Component Communication**:
  - Use shared hooks (e.g., `useGridLayout`) for layout logic.
  - Centralize theme logic in a context provider.

---

## Examples

### Example: Adding a New Component

1. Create the component in `app/components/`.
2. Use TailwindCSS for styling.
3. Export the component from an `index.ts` file for easy imports.

### Example: Adding a New Route

1. Define the route in `app/routes.ts`.
2. Create the route component in `app/routes/`.
3. Use dynamic imports to optimize loading.

---

## Best Practices

- **DRY Principle**: Refactor repeated logic into reusable hooks or utilities.
- **Accessibility**: Add ARIA tags and ensure sufficient color contrast.
- **Internationalization**: Use `react-i18next` or a similar library for multi-language support.
- **Testing**: Write unit tests for critical components and utilities.

---

For questions or clarifications, refer to the [README.md](../README.md) or consult the project maintainers.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marina-majden)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/marina-majden)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
