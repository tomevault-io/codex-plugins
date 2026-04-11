# Copilot Instructions - Landing Page Fiverr

## Project Overview
React 19 landing page built with Vite, TypeScript, and Styled Components. The project uses a theme-based styling approach with global styles and follows React functional component patterns.

## Tech Stack
- **React**: v19.2.0 (latest with new features)
- **TypeScript**: v5.9.3 (strict mode enabled)
- **Vite**: v7.2.4 (development server & bundler)
- **Styled Components**: Theme-based styling system
- **ESLint**: Flat config with React Hooks and React Refresh plugins

## Project Structure
```
src/
├── App.tsx          # Root component with ThemeProvider wrapper
├── main.tsx         # Entry point with React.StrictMode
├── assets/          # Static assets (images, icons)
└── style/           # Styling configuration
    ├── GlobalStyle.ts   # Global CSS reset & body styles
    └── theme.ts         # Theme configuration with color palette
```

## Key Patterns & Conventions

### Styling Architecture
- **Theme System**: All components access colors via `theme` prop from styled-components
- **Theme Colors**: `primary` (#007bff), `secondary` (#25d366), `dark`, `light`, `gray`, `text`
- **Global Reset**: Applied via `GlobalStyle` - universal box-sizing, zero margins/padding
- **Theme Access**: Use `${({ theme }) => theme.colors.property}` in styled components

Example from [GlobalStyle.ts](../src/style/GlobalStyle.ts):
```typescript
body {
  background: ${({ theme }) => theme.colors.light};
  color: ${({ theme }) => theme.colors.text};
}
```

### Component Structure
- Use functional components with TypeScript
- Wrap app with `ThemeProvider` from styled-components (see [App.tsx](../src/App.tsx))
- Always apply `GlobalStyle` inside `ThemeProvider`
- Note: Component folder structure is planned but not yet created (comment in App.tsx)

### TypeScript Configuration
- **Strict Mode**: Enabled with `noUnusedLocals`, `noUnusedParameters`
- **Module System**: ESNext with bundler resolution
- **JSX**: react-jsx transform (no React import needed)
- **Target**: ES2022 with DOM libraries

## Development Workflow

### Commands
```bash
npm run dev       # Start Vite dev server with HMR
npm run build     # Type check (tsc -b) + build for production
npm run lint      # Run ESLint on all files
npm run preview   # Preview production build locally
```

### Build Process
1. TypeScript compilation with project references (`tsc -b`)
2. Vite bundles for production (outputs to `dist/`)

### ESLint Configuration
- Uses flat config format (`eslint.config.js`)
- Ignores `dist/` directory globally
- Applies to `**/*.{ts,tsx}` files only
- Extends: `@eslint/js`, `typescript-eslint`, React Hooks, React Refresh

## When Creating Components
1. Create styled components using `styled-components` library
2. Import and use theme colors: `import { theme } from './styles/theme'`
3. Access theme in styled components via `theme` prop
4. Place new components in `src/` (component organization TBD)
5. Follow functional component pattern with TypeScript types

## Important Notes
- React 19 is used - leverage new features like automatic batching
- No React Compiler enabled (performance trade-off)
- Project uses Vite's Fast Refresh via `@vitejs/plugin-react`
- Entry point requires element with `id="root"` in [index.html](../index.html)
- Styled Components theme is centralized - don't hardcode colors

## Avoid
- Importing React for JSX (not needed with react-jsx transform)
- Hardcoding colors - always use theme
- Class components - use functional components only
- Ignoring TypeScript strict mode errors

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/veronicaschiller)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/veronicaschiller)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
