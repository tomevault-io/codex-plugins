# Copilot Instructions for Personal Website

## Project Overview
A modern, responsive portfolio website built with **Next.js 15** (App Router), **TypeScript**, **Tailwind CSS**, and custom React components. The site features a dark theme with glassmorphism effects, gradient animations, and an easter egg unlock sequence for a hidden music page.

## Architecture & Key Patterns

### Component Structure
- **Page Layout**: Single-page portfolio with sequential sections rendered in `src/app/page.tsx`
- **Section Components**: Modular section components (`Hero`, `About`, `Experience`, `Skills`, `Projects`, `Contact`, `Leadership`, `Education`) located in `src/components/`
- **Root Layout**: `src/app/layout.tsx` manages metadata, background gradient orbs, and `EasterEggNav` overlay
- **UI Component**: `GlassSurface` (`src/components/ui/GlassSurface.tsx`) is a client-side component with canvas-based glass morphism effects, distortion mapping, and complex props for customization

### Client vs Server Components
- Server-side by default (all section components are RSC)
- Only `EasterEggNav` uses `"use client"` for event listeners and router navigation
- `GlassSurface` uses `"use client"` for canvas manipulation and dark mode detection

### Design System
- **Colors**: Primary (cyan gradient: `#0ea5e9`), Accent (magenta: `#d946ef`), Orange (`#f97316`)
- **Custom Classes** (`src/app/globals.css`):
  - `.glass-card`: Semi-transparent background with backdrop blur
  - `.gradient-text`: Text with gradient fill (primary → accent → orange)
  - `.hover-glow`: Cyan glow effect on hover
  - `.gradient-border`: Animated border with gradient (currently blue/magenta)
- **Tailwind Extensions**: Custom animations (`fade-in`, `slide-up`, `float`) and extended color palette in `tailwind.config.ts`

### Path Aliases
- `@/*` resolves to `./src/*` for clean imports (e.g., `@/components/Hero`)

## Critical Developer Workflows

### Development & Build
```bash
npm run dev        # Start dev server on http://localhost:3000
npm run build      # Production build
npm run start      # Run production server
npm run lint       # ESLint check (Next.js config)
```

### Adding New Sections
1. Create component in `src/components/` (e.g., `NewSection.tsx`)
2. Make it a Server Component by default (no `"use client"`)
3. Import and add to `src/app/page.tsx`
4. Use `.glass-card`, `.gradient-text`, and custom animations for consistency

### Styling Approach
- **Tailwind first**: Utility classes for layout, spacing, responsive design
- **Custom CSS**: Only in `src/app/globals.css` for shared effects (gradients, animations, scrollbars)
- **Avoid inline styles**: Use Tailwind classes unless dynamic values require `style` prop

## Project-Specific Conventions

### Easter Egg System
- `EasterEggNav` (`src/components/EasterEggNav.tsx`) listens for key sequence `m-u-s-i-c`
- On completion: plays audio from `/audio/ps3-orchestra.mp3`, sets `sessionStorage.musicUnlocked`, navigates to `/music`
- The music page exists but is conditionally gated; ensure audio file is present in `public/audio/`

### Metadata & SEO
- Managed in `src/app/layout.tsx` with OpenGraph support
- Update title, description, and keywords when personalizing

### Image Optimization
- Uses Next.js `Image` component (see Hero profile photo)
- Images in `public/` (e.g., `profile.jpg`)

## Integration Points & Dependencies

### Key Libraries
- **next** (15.1.6): App Router, image optimization, font loading
- **react** (19.0.0): Components with hooks
- **tailwindcss** (3.4.17): Utility CSS framework
- **typescript** (5.7.3): Type safety

### No External Headless UI
- Custom components and Tailwind utilities; no shadcn/ui or similar dependencies
- Canvas-based `GlassSurface` built from scratch for advanced effects

### Audio Handling
- `EasterEggNav` creates Audio element dynamically (window-only)
- Volume set to 0.35; error handling with `.catch()` for autoplay restrictions

## Common Modifications

### Update Personal Info
- Hero section text: `src/components/Hero.tsx`
- Experience/Education/Skills: Respective component files
- Metadata: `src/app/layout.tsx`

### Customize Colors
- Gradient accent: Update `from-primary-500 to-accent-500` Tailwind classes
- Global styles: Modify gradient stops in `src/app/globals.css`
- Theme: Edit color scales in `tailwind.config.ts`

### Add New Pages
- Create route in `src/app/` (e.g., `src/app/newpage/page.tsx`)
- Update root layout if global styling needed
- Use path alias `@/components` for imports

### GlassSurface Customization
- Props: `width`, `height`, `blur`, `opacity`, `distortionScale`, `distortion`, color channels
- Example usage in sections showing glassmorphism effects

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Aristicrat) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
