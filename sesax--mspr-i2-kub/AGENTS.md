# Copilot Instructions

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

## Tech Stack

This project uses the following technologies:

- **Next.js 14.0.0** - React framework with App Router and server-side rendering
- **React 18.2.0** - JavaScript library for building user interfaces
- **TypeScript** - Typed superset of JavaScript
- **TailwindCSS 3.3.0** - Utility-first CSS framework
- **Framer Motion 10.18.0** - Animation library for React
- **ESLint** - Code linting and formatting

## Project Structure

- `/src/app/` - Next.js App Router pages and layouts
- `/src/components/` - Reusable React components
- `/src/components/ui/` - UI-specific components
- `/public/` - Static assets

## Development Guidelines

1. **Components**: Use TypeScript for all components with proper type definitions
2. **Styling**: Use TailwindCSS utility classes for styling
3. **Animations**: Use Framer Motion for animations and transitions
4. **Client Components**: Add 'use client' directive when using browser-specific APIs or interactive features
5. **Responsiveness**: Ensure all components work well on mobile and desktop

## Code Style

- Use functional components with React hooks
- Prefer named exports over default exports for utilities
- Use meaningful component and variable names
- Keep components small and focused on a single responsibility
- Use proper TypeScript interfaces for props

## Animation Patterns

- Use `initial`, `animate`, and `transition` props for entrance animations
- Use `whileHover` and `whileTap` for interaction feedback
- Use `whileInView` for scroll-triggered animations
- Keep animations smooth and purposeful

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Sesax) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
