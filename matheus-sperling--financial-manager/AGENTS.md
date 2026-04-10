# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

```bash
# Setup
npm install

# Development server with hot reload on http://localhost:3000
npm run dev

# Build for production (Next.js static export)
npm run build

# Start production server (after build)
npm run start

# Alternative build command (same as build)
npm run export

# Preview production build (same as start)
npm run preview

# Code quality - ALWAYS run before committing (migrated to ESLint CLI)
npm run lint

# Fix linting issues automatically
npm run lint:fix

# Type checking (available but not in automated workflow)
npm run type-check

# Tests (no tests configured)
npm test
```

## 🚢 Deployment to GitHub Pages

This project is configured for **automatic deployment to GitHub Pages** using GitHub Actions with Next.js static export.

### Automatic Deployment

Deployment happens automatically on every push to the `main` branch via GitHub Actions workflow (`.github/workflows/nextjs.yml`):

1. **Push to main branch** - triggers the deployment workflow
2. **GitHub Actions builds** - runs `next build` and creates static files in `dist/`
3. **Automatic deployment** - GitHub Actions deploys to GitHub Pages
4. **Live site updated** - changes are live at `https://matheus-sperling.github.io/Financial-Manager/`

### Manual Deployment (if needed)

You can also trigger deployment manually:
- Go to repository **Actions** tab
- Select "Deploy Next.js site to Pages" workflow  
- Click "Run workflow"

**Current Configuration**: This repository deploys to `https://matheus-sperling.github.io/Financial-Manager/`

**Forking**: If you fork this repository, **no configuration changes needed** - GitHub Actions automatically handles the repository name and base paths.

## Project Architecture

This is a modern React application built with TypeScript, Next.js, Tailwind CSS, and shadcn/ui components. It features excellent mobile responsiveness, static export compatibility, and follows modern web development best practices.

### Key Architecture Patterns

- **React 19**: Latest React with modern hooks and state management
- **Next.js 15**: App Router with static export for GitHub Pages compatibility
- **TypeScript**: Type-safe development with excellent IntelliSense
- **Tailwind CSS**: Utility-first CSS framework for rapid UI development
- **shadcn/ui**: Beautifully designed, accessible, and customizable components
- **Client-Side Storage**: Custom useClientStorage hook for SSR-compatible localStorage handling
- **Mobile-First Design**: Responsive design optimized for all device sizes
- **Data Persistence**: All data automatically saved to browser localStorage with hydration support

### Core Application Structure

```
app/
├── layout.tsx              # Next.js root layout with metadata and fonts
├── page.tsx                # Next.js page component (entry point)
└── globals.css             # Global styles with Tailwind imports

src/
├── components/
│   ├── ui/                 # shadcn/ui base components
│   │   ├── button.tsx      # Button component with variants (consolidated)
│   │   ├── card.tsx        # Card component
│   │   ├── dialog.tsx      # Dialog/modal component
│   │   └── input.tsx       # Input component
│   ├── AppContent.tsx      # Main application logic and UI
│   ├── DebtSummary.tsx     # Financial summary cards component
│   ├── DebtTable.tsx       # Responsive debt table/card component
│   └── ReorderButton.tsx   # Mobile-friendly select-and-move reordering
├── hooks/
│   ├── useDebtManager.ts   # Custom hook for complete state management
│   └── useClientStorage.ts # SSR-compatible localStorage hook with hydration
├── lib/
│   └── utils.ts            # Utility functions and helpers (cn, clsx)
└── types/
    └── debt.ts             # TypeScript interfaces (Person, Debt, DebtManagerState)

next.config.js              # Next.js configuration for static export
tailwind.config.js          # Tailwind CSS configuration
tsconfig.json              # TypeScript configuration with path aliases (@/ -> src/)
postcss.config.js           # PostCSS configuration for Tailwind
package.json                # Dependencies and build scripts
```

### Key Application Features

- **Person & Debt Management**: Add/edit/remove people and their associated debts
- **Hidden Debts**: Mark debts as hidden to exclude from total calculations
- **Debt Actions**: Edit, duplicate, and remove individual debts with confirmations
- **Select-and-Move Reordering**: Mobile-friendly reordering system replacing drag-and-drop
- **Currency Support**: Toggle between USD ($) and BRL (R$) with automatic formatting
- **Themes**: Light/dark mode toggle with automatic persistence using data-theme attribute
- **Internationalization**: English/Portuguese with browser language detection and flag toggles
- **Responsive Design**: Mobile-first approach using Tailwind responsive utilities with card-based mobile layout
- **Data Persistence**: All data automatically saved to browser localStorage with SSR-compatible hydration
- **Static Export**: Full Next.js static export compatibility for GitHub Pages deployment

## Development Workflow

1. Make changes to React components, hooks, or styles
2. Changes reflect immediately with Next.js Fast Refresh on `http://localhost:3000`
3. **CRITICAL**: Always run `npm run lint` before committing
4. **Manual testing required**: Test all modified features in browser
5. Verify data persistence by refreshing the page
6. Test static export builds with `npm run build`

### Manual Testing Scenarios

Always test these scenarios after making changes:

- Add/edit/remove persons and debts
- Duplicate debts and verify functionality
- Toggle hidden debts and verify total calculations exclude hidden amounts
- Test select-and-move reordering on mobile and desktop (both people and debts)
- Switch between currencies (USD/BRL) and verify formatting with proper symbols
- Toggle theme (light/dark) and verify persistence across page refreshes
- Switch languages (EN/PT) and verify all translations are applied
- Test responsive design on different screen sizes (desktop table vs mobile cards)
- Clear all data and verify confirmation dialog works
- Refresh page to verify localStorage data persistence and hydration
- Test that confirmation dialogs appear for destructive actions

## Code Style & Patterns

- **ESLint Configuration**: Uses modern eslint.config.mjs with TypeScript-aware rules, migrated from deprecated next lint
- **TypeScript**: Strict type checking enabled, interfaces defined in types/debt.ts
- **Next.js Patterns**: App Router structure, server/client component separation
- **State Management**: useDebtManager hook centralizes all application state and actions
- **SSR Compatibility**: useClientStorage hook handles localStorage with proper hydration
- **Component Architecture**: Small, focused components with clear prop interfaces
- **Styling**: Tailwind CSS classes with shadcn/ui component system

## Important Implementation Details

- **No Backend**: Pure frontend React application with Next.js static export
- **Build Process**: Next.js static export generates static HTML/CSS/JS for GitHub Pages
- **Data Storage**: Client-side localStorage with SSR-compatible hydration patterns
- **State Management**: Custom useDebtManager hook manages all application state
- **Theme System**: data-theme attribute switching with Tailwind dark: modifier
- **Internationalization**: Translation objects with browser language detection
- **Path Aliases**: @/ alias configured for src/ directory in tsconfig.json
- **Static Export**: next.config.js configured for GitHub Pages with automatic GitHub Actions deployment

## Frequently Modified Files

- **src/hooks/useDebtManager.ts**: Core state management, adding new features, data persistence logic
- **src/hooks/useClientStorage.ts**: localStorage integration with SSR compatibility
- **src/components/AppContent.tsx**: Main component with translations, UI layout, theme/language/currency toggles
- **src/components/DebtTable.tsx**: Person and debt management UI, form interactions
- **src/components/DebtSummary.tsx**: Financial summary display and calculations
- **src/components/ReorderButton.tsx**: Mobile-friendly reordering functionality
- **src/components/ui/button.tsx**: Button component with consolidated variants (no separate button-variants.ts)
- **src/components/ui/**: Other base shadcn/ui components (card, dialog, input)
- **src/types/debt.ts**: When adding new TypeScript interfaces or modifying existing ones
- **app/layout.tsx**: Site metadata, fonts, and global layout configuration
- **app/globals.css**: Global styles, Tailwind imports, and custom CSS
- **next.config.js**: Static export configuration for automated GitHub Actions deployment
- **.github/workflows/nextjs.yml**: GitHub Actions workflow for automatic deployment

## Critical Data Persistence Requirements

**I don't want ANY changes to make me lose my saved data on the browser. Always test data persistence after changes.**

- All data is stored in localStorage using keys defined in useDebtManager.ts
- Test data persistence by refreshing the page after making changes
- The useClientStorage hook handles localStorage with proper SSR hydration
- Client-side hydration ensures data loads correctly after Next.js static generation
- Never modify localStorage keys without migration strategy
- Always test in both development and production builds

## Always update copilot-instructions.md, README.md, and this file (CLAUDE.md) with any significant changes to architecture, commands, or deployment process.

## GitHub Actions Deployment

The project uses automated GitHub Actions deployment with these key files:
- `.github/workflows/nextjs.yml` - Main deployment workflow
- `next.config.js` - Next.js configuration for static export
- No manual deployment commands needed - deployment is fully automated on push to main

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/matheus-sperling)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/matheus-sperling)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
