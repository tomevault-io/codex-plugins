# HDNCare Codebase Instructions

## Project Overview
HDNCare is a React + Vite beauty/salon appointment booking application with a focus on stylist discovery and appointment management. It's a modern frontend-first SPA with Tailwind CSS styling and Headless UI components.

## Architecture & Key Patterns

### Tech Stack
- **Build**: Vite with Rolldown, React 19
- **Routing**: React Router v7 with path-based pages under `src/pages/`
- **Styling**: Tailwind CSS 3.4 with @tailwindcss/forms plugin; all styles in `src/index.css`
- **UI Components**: Headless UI, React Icons, Heroicons
- **Animations**: Framer Motion (present but not yet heavily utilized)

### Project Structure
- `src/pages/` - Route pages: Home, Stylists, Login, About, Contact, MyProfile, MyAppointments, Appointments
- `src/components/` - Reusable components like Navbar
- `src/assets/` - Images and logos imported via JS modules
- Routes defined in App.jsx: supports parameterized paths like `/stylists/:speciality` and `/appointments/:styId`

### Component Patterns
1. **Functional Components**: All components are functional with React hooks (useState shown in App.jsx)
2. **Navbar Pattern**: [Navbar.jsx](../src/components/Navbar.jsx) imports assets as objects (`{assets}`) and uses NavLink wrappers - refactor to add proper route paths to NavLink elements
3. **Asset Management**: Assets imported from asset modules (e.g., `assets.logo`) - maintain this pattern for centralized management

### Data & State
- Currently minimal state management; suitable for context API or light state libraries as app scales
- Routes indicate data models: Stylists (with specialties), Appointments, Profiles
- No API integration yet visible; plan for REST/GraphQL endpoints for backend sync

## Development Workflow

### Commands
```bash
npm run dev          # Start Vite dev server with HMR
npm run build        # Production build to dist/
npm run lint         # ESLint check (configured in eslint.config.js)
npm run preview      # Preview production build locally
```

### ESLint Configuration
- Enforces recommended rules + React Hooks plugin + React Refresh
- Rule: `varsIgnorePattern: '^[A-Z_]'` allows uppercase/snake_case unused variables (common for React components and constants)
- Remember to import React when needed for older JSX syntax (not required in React 19)

## Critical Conventions

1. **Tailwind First**: Use `mx-4 sm:mx-[10%]` responsive padding pattern (seen in App.jsx) - this is the project's standard responsive wrapper
2. **CSS Classes**: All Tailwind utility classes in JSX; avoid inline styles
3. **Asset Imports**: Centralize images/logos in `src/assets/` and import as named objects - not as direct file paths
4. **Route Paths**: Always include route paths in NavLink/Link components (Navbar currently missing these)
5. **File Naming**: PascalCase for components (Navbar.jsx), lowercase for utilities/contexts

## Common Tasks

- **Add New Page**: Create file in `src/pages/`, add Route in App.jsx, link via Navbar
- **Add Component**: Create in `src/components/`, import where needed
- **Style Changes**: Add Tailwind classes to JSX elements; modify `tailwind.config.js` for custom theme extensions (currently none)
- **Add Assets**: Place in `src/assets/`, export from assets module, import as `{assets.name}`

## Known Issues & Gaps
- Navbar NavLink elements missing path props
- Assets structure suggests centralized module but not yet examined - verify `src/assets/assets.js` exports
- No backend API wiring; appointment/stylist data is static placeholder
- State management pattern TBD as data flows scale

## Git Workflow
Standard git workflow assumed from `.git` presence; commit messages and branch strategy not specified.

---
> Source: [thjp197/HDNCare](https://github.com/thjp197/HDNCare) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-19 -->
