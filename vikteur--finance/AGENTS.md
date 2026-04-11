# Copilot Instructions for AI Coding Agents

## Project Overview

This project is a React frontend application built with Vite, designed for maintainability, scalability, and testability. It leverages Vite's fast HMR (Hot Module Replacement) and build optimizations, uses reusable React components, follows clean architecture principles, and employs Jest for unit and integration testing. These instructions are tailored for development on Windows.

## Directory Structure

- `/src/` — Main source code
  - `/components/` — Reusable UI components (atomic, well-documented)
  - `/features/` — Feature modules (business logic, state, API calls)
  - `/hooks/` — Custom React hooks
  - `/utils/` — Utility functions
  - `/styles/` — Global and modular styles
  - `/tests/` — Standalone test utilities/mocks (if not colocated)
- `/public/` — Static assets
- `/__mocks__/` — Jest mocks (if needed)

## Clean Architecture Principles

- **Separation of Concerns:**
  - UI, business logic, and data access are strictly separated.
  - Components are presentational; state and logic live in feature modules or hooks.
- **Dependency Inversion:**
  - Features depend on abstractions (interfaces/types), not concrete implementations.
- **Testability:**
  - All business logic is unit-testable and decoupled from UI.
- **Reusability:**
  - Components are atomic, stateless when possible, and documented with PropTypes or TypeScript.

## Developer Workflows

### Setup

1. Install dependencies:
   ```cmd
   npm install
   ```
2. Start development server:
   ```cmd
   npm start
   ```
3. Run tests:
   ```cmd
   npm test
   ```
4. Build for production:
   ```cmd
   npm run build
   ```

### Quick commands

- Start Vite dev server: `npm run dev` (includes fast HMR)
- Build production: `npm run build`
- Preview build: `npm run preview`
- Run tests: `npm test` (watch: `npm run test:watch`)
- Run lint: `npm run lint`
- Format code: `npm run format`
- Run full CI locally: `npm ci && npm run lint && npm test`

### Key files & configs

- `package.json` — scripts and dependencies
- `vite.config.js` — Vite configuration (plugins, build options, dev server)
- `jest.config.js` — Jest configuration
- `src/setupTests.js` — test setup and global mocks
- `.eslintrc.js` / `.eslintrc.json` — ESLint rules (include `jsx-a11y` rules)
- `tsconfig.json` (if TypeScript)
- `index.html` — Entry point for Vite

### Accessibility & testing

- Use `eslint-plugin-jsx-a11y` for lint checks and `jest-axe` / `axe-core` for automated accessibility tests in CI.
- Add an accessibility smoke test that runs `jest-axe` against critical pages/components.
- Ensure all interactive elements have keyboard focus states and ARIA attributes where semantic HTML isn't sufficient.

### Testing

- Use Jest for all unit and integration tests.
- Test files colocated as `ComponentName.test.js` or in `/tests/`.
- Use React Testing Library for component rendering and interaction.
- Mocks and test utilities go in `/__mocks__/` or `/tests/`.

### Linting & Formatting

- Use ESLint and Prettier for code quality and formatting.
- Run lint:
  ```cmd
  npm run lint
  ```

## Conventions

- Use functional components and hooks.
- Prefer TypeScript for type safety (if enabled).
- Document all public APIs (components, hooks, utilities).
- Avoid direct DOM manipulation; use React idioms.
- Keep business logic out of components.
- Use environment variables via `.env` for config.

## Integration Points

- State Management: Use React Context or Redux (if needed), but keep state local when possible.
- Styles: Use CSS Modules, styled-components, or Tailwind (document choice).

## Vite Configuration & Best Practices

### Development Features

- **Hot Module Replacement (HMR)**:
  - Components auto-update without page refresh
  - State preservation during component updates
  - CSS/SCSS hot reload support

### Build Optimization

- **Dynamic Imports**: Use for route-based code splitting
  ```jsx
  const MyComponent = React.lazy(() => import("./MyComponent"));
  ```
- **Asset Handling**:
  - Static assets under `/public/` are served at `/`
  - Import assets directly in components for build optimization
  ```jsx
  import logo from "./assets/logo.svg";
  ```

### Performance Optimizations

- Use `React.lazy()` for route components
- Enable build features in `vite.config.js`:
  ```js
  export default defineConfig({
    build: {
      minify: "terser",
      rollupOptions: {
        output: {
          manualChunks: {
            vendor: ["react", "react-dom"],
          },
        },
      },
    },
  });
  ```

### Development Commands

- `npm run dev` - Start dev server with HMR
- `npm run build` - Production build
- `npm run preview` - Preview production build locally
- `npm run serve` - Serve production build

### Common Issues & Solutions

- **Path Aliases**: Configure in both `vite.config.js` and `tsconfig.json`
- **CSS Modules**: Filename pattern `*.module.css`
- **SVG Imports**: Use `vite-plugin-svgr` for React components

## State Management Guide

This guide follows React best practices and includes:

- Practical examples with real-world use cases
- Clear code samples that demonstrate each pattern
- Testing strategies for state management
- Guidelines for when to use each approach

The instructions maintain clean architecture principles by:

- Keeping state close to where it's used
- Separating concerns between UI and state management
- Providing clear patterns for testing
- Following the project's established directory structure

### Local State Management

```jsx
// Prefer local state for component-specific data
function Counter() {
  const [count, setCount] = useState(0);
  return <button onClick={() => setCount(count + 1)}>{count}</button>;
}
```

### Custom Hooks (Feature-Level State)

```jsx
// src/features/auth/useAuth.js
export function useAuth() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  const login = useCallback(async (credentials) => {
    setLoading(true);
    try {
      const user = await loginAPI(credentials);
      setUser(user);
    } finally {
      setLoading(false);
    }
  }, []);

  return { user, loading, login };
}
```

### Context for Shared State

```jsx
// src/features/theme/ThemeContext.jsx
export const ThemeContext = createContext();

export function ThemeProvider({ children }) {
  const [theme, setTheme] = useState("light");

  const toggleTheme = useCallback(() => {
    setTheme((t) => (t === "light" ? "dark" : "light"));
  }, []);

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}
```

### State Management Decision Tree

1. **Component State (`useState`)**

   - Use for: UI state, form inputs, toggles
   - When: State is only used by one component
   - Example: Modal open/closed, input values

2. **Custom Hooks**

   - Use for: Reusable logic, API calls, complex state
   - When: Logic is shared between components
   - Location: `/src/hooks/` or `/src/features/[feature]/hooks/`

3. **Context API**

   - Use for: Theme, auth, shared configurations
   - When: State needs to be accessed by many components
   - Avoid: Frequent updates that affect many components

4. **External State Management (Redux/Zustand)**
   - Use for: Complex global state, heavy data manipulation
   - When:
     - Multiple related state updates
     - Complex state interactions
     - Performance optimization needed
   - Location: `/src/store/` with feature-based slices

### Testing State

```jsx
// Example test for custom hook
import { renderHook, act } from "@testing-library/react-hooks";

test("useCounter increments count", () => {
  const { result } = renderHook(() => useCounter());

  act(() => {
    result.current.increment();
  });

  expect(result.current.count).toBe(1);
});
```

## Example Component Pattern

```jsx
// src/components/Button/Button.jsx
import React from "react";
import PropTypes from "prop-types";

export function Button({ children, onClick }) {
  return <button onClick={onClick}>{children}</button>;
}
Button.propTypes = {
  children: PropTypes.node.isRequired,
  onClick: PropTypes.func,
};
```

---

_Update this file as the project evolves. Document new conventions, workflows, and architectural decisions for future contributors and AI agents._

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Vikteur)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Vikteur)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
