# AI Coding Agent Instructions

## Project Overview
This is Keith Lim's personal portfolio website - a single-page application built with React + Vite, deployed to GitHub Pages at `keithfykai.github.io/keithfykai/`. The project showcases a Computer Science student's work, experience, and projects.

## Tech Stack
- **Framework**: React 19.1.0 with Vite 6.3.5 for development and build
- **Styling**: Tailwind CSS 4.1.6 (imported via `@import "tailwindcss"` in CSS files)
- **Routing**: React Router DOM 7.6.0 (using BrowserRouter)
- **UI Libraries**: Bootstrap 5.3.6, Flowbite 3.1.2, React Icons, Headless UI, Heroicons
- **Language**: JavaScript (JSX), ES2020+

## Project Structure
```
portfolio/
├── src/
│   ├── Components/       # Page components (Hero, Navbar, Projects, etc.)
│   ├── assets/          # Images and static assets
│   ├── App.jsx          # Main router setup
│   ├── main.jsx         # React entry point
│   └── index.css        # Tailwind import
├── vite.config.js       # Base URL: '/keithfykai/'
└── eslint.config.js     # Flat config format
```

## Key Configuration Details

### GitHub Pages Deployment
- **Base path**: `/keithfykai/` (set in `vite.config.js`)
- **Deploy command**: `npm run deploy` (builds and pushes to gh-pages branch)
- **Critical**: All routes in `App.jsx` must start with `/keithfykai/` (e.g., `<Route path="/keithfykai/" element={<Hero />} />`)

### Component Patterns
1. **Named exports**: All components use named exports (`export function ComponentName()`)
2. **Functional components**: No class components, all use function syntax
3. **React Router Links**: Use `<Link to="/path">` instead of `<a href>`, imported from `react-router-dom`
4. **Tailwind-first styling**: Use Tailwind utility classes directly in JSX
5. **Mobile-first responsive**: Components use responsive breakpoints (`md:`, `lg:`, `sm:`)

### Styling Conventions
- Tailwind utilities applied inline (e.g., `className="mx-auto max-w-2xl text-center"`)
- No CSS modules or styled-components
- Import Tailwind via `@import "tailwindcss"` in CSS files (not `@tailwind` directives)
- Responsive design: Hidden mobile menu toggle (`md:hidden`), desktop nav (`hidden md:flex`)

### ESLint Rules
- **Flat config** format (ESLint 9.x)
- Unused vars allowed if uppercase/underscored: `varsIgnorePattern: '^[A-Z_]'`
- React Hooks rules enforced
- React Fast Refresh warnings enabled

## Development Workflow

### Common Commands
```bash
cd portfolio
npm run dev          # Start dev server
npm run build        # Build for production
npm run preview      # Preview production build
npm run deploy       # Build and deploy to GitHub Pages
npm run lint         # Run ESLint
```

### Adding New Pages
1. Create component in `src/Components/NewPage.jsx` using named export
2. Add route in `App.jsx`: `<Route path="/newpage" element={<NewPage />} />`
3. Update navigation in `Navbar.jsx` with `<Link to="/newpage">`
4. Ensure responsive mobile menu includes new link with `onClick={closeMenu}`

### Working with Assets
- Place images in `src/assets/`
- Import using relative paths: `import photo from '../assets/photo.jpg'`
- Use in JSX: `<img src={photo} alt="description" />`

## Project-Specific Conventions

### Navigation Structure
- Homepage route: `/keithfykai/` (not just `/`)
- Other routes: `/aboutme`, `/projects`, `/contactme` (relative to base)
- 404 fallback: `<Route path="*">` catches undefined routes

### State Management
- Local component state with `useState` hooks
- Example: Mobile menu toggle in `Navbar.jsx` uses `isMobileMenuOpen` state
- No global state management (no Redux/Zustand/Context in use)

### Content Organization
- Project showcase data is hardcoded in `Projects.jsx` component
- External links use regular `<a>` tags; internal navigation uses `<Link>`
- Photos and assets referenced via imports, not public folder

## Common Pitfalls to Avoid
1. **Don't use `<a href>` for internal navigation** - breaks SPA routing, use `<Link to>`
2. **Don't forget base path** - All routes must account for `/keithfykai/` base
3. **Don't use default exports** - Project uses named exports consistently
4. **Don't use old Tailwind directives** - Use `@import "tailwindcss"`, not `@tailwind base/components/utilities`
5. **Don't skip mobile menu handler** - New nav links need `onClick={closeMenu}` in mobile view

## External Dependencies to Note
- **Vite plugin**: Uses `@vitejs/plugin-react` (Babel-based Fast Refresh)
- **Tailwind Vite plugin**: `@tailwindcss/vite` v4.x (newer integration)
- **gh-pages**: Automated deployment utility for GitHub Pages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/keithfykai)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/keithfykai)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
