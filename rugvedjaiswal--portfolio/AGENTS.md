# Rugved Jaiswal's Portfolio - AI Agent Instructions

## Project Overview
A modern, responsive personal portfolio website built with **React + Tailwind CSS**, deployed on **Vercel**. Showcases Rugved's skills, education, projects, and professional background to recruiters and collaborators.

**Tech Stack**: React.js, Tailwind CSS, React Icons, JavaScript ES6+  
**Deployment**: Vercel (automatic on `main` branch push)  
**Live**: https://rugved-jaiswal-portfolio.vercel.app

---

## Architecture Overview

### Component Structure
```
App.jsx (Main container)
├── Navbar (Fixed header with navigation & mobile menu)
├── Home (Hero section with CTA buttons)
├── About (Bio, background, location)
├── Skills (Tech categories + proficiency bars)
├── Education (Timeline with degrees, certifications)
├── Projects (Featured projects grid, coming soon)
├── Contact (Contact form + social links)
└── Footer (Copyright, social links, Vercel note)
```

### Key Design Patterns
- **Section-based Layout**: Each page section (`/pages/`) scrolls smoothly with fixed navbar
- **Reusable Styles**: Tailwind utility classes with custom `@layer components` for buttons, sections in `src/styles/index.css`
- **State Management**: Minimal (React hooks for navbar menu toggle, contact form state)
- **Responsive First**: Mobile-first design with `md:` and `lg:` breakpoints throughout

---

## Developer Workflows

### Local Development
```bash
npm install          # Install dependencies (one-time)
npm start           # Start dev server on http://localhost:3000
npm run build       # Build for production (creates /build directory)
```

### Deployment to Vercel
- **Automatic**: Push to `main` branch → Vercel auto-builds and deploys
- **Manual**: `vercel deploy` (if Vercel CLI installed)
- **Rollback**: Vercel dashboard → Deployments → Promote previous build

### Adding New Content

**Add a Project**:
1. Update `src/pages/Projects.jsx` - Add object to `projects` array
2. Include: `title`, `description`, `technologies` (array), `github`, `link`
3. Push to main → Auto-deploys

**Update Skills**:
1. Edit `src/pages/Skills.jsx` - Modify `skillCategories` array
2. Add/remove skill objects with `title` and `skills` array
3. Push to main → Auto-deploys

**Update Education**:
1. Edit `src/pages/Education.jsx` - Modify `educationData` array
2. Include: `degree`, `institution`, `year`, `percentage` (if applicable), `details` array

---

## Project-Specific Conventions

### File Organization
- **Components** (`/components`): Reusable UI parts (Navbar, Footer) - have internal state (e.g., mobile menu)
- **Pages** (`/pages`): Full section components (Home, About, Skills, etc.) - mostly presentational
- **Styles** (`/styles/index.css`): Global Tailwind directives + custom component utilities

### Color Scheme
- **Primary Background**: `bg-slate-900` (dark navy)
- **Secondary Background**: `bg-slate-800` (slightly lighter)
- **Accent Color**: `text-blue-400` or `bg-blue-600` (bright blue)
- **Text**: `text-white` (main), `text-slate-300/400` (secondary), `text-slate-500` (tertiary)

### Tailwind Class Patterns
- **Buttons**: Use `btn-primary` or `btn-secondary` classes from `index.css`
- **Sections**: Wrap in `section` class + `container-max` for max-width and centering
- **Hover Effects**: Use `hover:shadow-lg hover:shadow-blue-500/20` for subtle glow on cards
- **Transitions**: `transition-all duration-300` for smooth animations

### Responsive Breakpoints
- **Mobile First**: Base styles apply to mobile
- **`md:` (768px+)**: Tablet and desktop changes (2-column grids, flex instead of block)
- **`lg:` (1024px+)**: Large desktop enhancements (3-column grids)

---

## Integration Points & External Links

### Social Media & Contact
- **GitHub**: `https://github.com/rugvedjaiswal`
- **LinkedIn**: `https://www.linkedin.com/in/rugved-jaiswal-2303x`
- **Email**: `jaiswalrn@rknec.edu`
- **Phone**: `+91 7796280726`

All links use `target="_blank" rel="noopener noreferrer"` for external URLs.

### Vercel Deployment Config
- **`vercel.json`**: Specifies `buildCommand: "npm run build"` and `outputDirectory: "build"`
- **Auto-detected**: Vercel recognizes React app via `package.json`
- **No API Backend**: Contact form currently frontend-only (shows success message after 3 seconds)

---

## Critical Files & Patterns

| File | Purpose | Key Patterns |
|------|---------|--------------|
| `src/App.jsx` | Main component, imports all pages | Renders all sections vertically |
| `src/styles/index.css` | Global styles, Tailwind directives | Custom `@layer components` for DRY utilities |
| `src/pages/Home.jsx` | Hero section | `animate-bounce` scroll indicator, grid stats |
| `src/pages/Skills.jsx` | Tech skills showcase | `skillCategories` array, progress bar visualization |
| `src/pages/Contact.jsx` | Contact form & info | Form state with `useState`, email links |
| `package.json` | Dependencies & build config | React, Tailwind, React Icons |
| `vercel.json` | Deployment settings | Build output directory configuration |

---

## Common Tasks & Examples

### Change Accent Color Globally
Update `tailwind.config.js` theme colors, then replace `text-blue-400` / `bg-blue-600` instances in components.

### Add a New Section
1. Create `src/pages/NewSection.jsx` with `id="newsection"` in section tag
2. Import in `App.jsx`: `import NewSection from './pages/NewSection'`
3. Add to Navbar links in `Navbar.jsx`
4. Push to main → Vercel deploys

### Modify Contact Form Submission
Currently shows frontend success message. To add backend:
1. Update form `onSubmit` handler in `Contact.jsx` to call API endpoint
2. Create Spring Boot backend (Phase 2 in PRD)
3. Update env variables in Vercel dashboard

---

## Performance & Optimization

- **Lazy Loading**: Not critical for this size; consider if adding many images
- **CSS**: Tailwind purges unused styles in production (`npm run build`)
- **Images**: Use emoji placeholders instead of image files (fast load)
- **Lighthouse Target**: ~95+ score maintained by minimal dependencies

---

## Future Enhancements (Roadmap)

- Backend contact form (Spring Boot API)
- Blog section with markdown support
- Project case studies with images
- Dark/Light mode toggle
- Analytics integration
- Custom domain setup

---

## Quick Reference

**Add content**: Update array in corresponding page file → push to main  
**Deploy**: Automatic on push to `main`; manual via Vercel dashboard  
**Test locally**: `npm start` opens http://localhost:3000  
**Build production**: `npm run build` creates `/build` directory  
**Debug styles**: Inspect in browser DevTools; check Tailwind class names in components

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rugvedjaiswal) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
