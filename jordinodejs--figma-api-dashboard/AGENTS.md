# Figma API Dashboard - GitHub Copilot Custom Instructions

## Project Overview

This is a Next.js 15 application with TypeScript, Tailwind CSS 4, and Figma integration features. The project follows modern React patterns with App Router and server components.

## Technology Stack

- **Framework**: Next.js 15.3.4 with App Router and Turbopack
- **Language**: TypeScript 5+ with strict type checking
- **Styling**: Tailwind CSS 4 with PostCSS integration
- **React**: React 19 with latest patterns and hooks
- **Package Manager**: Support for npm, yarn, pnpm, and bun

## Development Guidelines

### Code Standards

- Use TypeScript for all components and utilities with explicit type annotations
- Prefer Server Components by default; use Client Components only when necessary with "use client" directive
- Follow Next.js 15 App Router conventions and file structure
- Use modern CSS-in-JS patterns with Tailwind CSS 4 syntax
- Implement proper error boundaries and loading states using app/error.tsx and app/loading.tsx

### Component Patterns

- Create reusable components in src/components/ directory
- Use React 19 features like concurrent features and server actions
- Implement proper prop validation with TypeScript interfaces
- Follow composition over inheritance patterns
- Use Server Actions for form handling and data mutations

### Styling Guidelines

- Use Tailwind CSS 4 utility classes consistently
- Implement responsive design with mobile-first approach
- Follow design system principles with consistent spacing and typography
- Use CSS variables for theming and dark mode support
- Prefer Tailwind's built-in classes over custom CSS

### Performance Optimization

- Utilize Next.js Image component for optimized images
- Implement proper code splitting with dynamic imports
- Use Server Components for data fetching when possible
- Optimize bundle size with proper tree shaking
- Implement caching strategies for API routes and data fetching

### File Naming Conventions

- Use kebab-case for directories and files
- Component files should use PascalCase with .tsx extension
- Use descriptive names that reflect component purpose
- Group related components in feature-based directories

### Data Fetching

- Use Server Components for initial data loading
- Implement Server Actions for mutations and form submissions
- Use proper error handling with try-catch blocks
- Implement loading states and error boundaries
- Cache data appropriately using Next.js caching mechanisms

### Figma Integration

- Implement Figma API integration patterns following best practices
- Use proper authentication and authorization for Figma API calls
- Handle Figma design tokens and component synchronization
- Implement proper error handling for Figma API interactions

### Security Practices

- Never expose sensitive API keys or secrets to the client
- Use environment variables with NEXT*PUBLIC* prefix only for public data
- Implement proper input validation and sanitization
- Use Content Security Policy (CSP) headers
- Follow OWASP security guidelines for web applications

### Testing and Quality

- Write unit tests for components and utilities
- Implement integration tests for critical user flows
- Use ESLint with Next.js recommended configuration
- Follow accessibility best practices (WCAG guidelines)
- Implement proper SEO with metadata API

### Development Workflow

- Use TypeScript strict mode for enhanced type safety
- Run `npm run dev` with Turbopack for fast development
- Use proper Git commit conventions and pull request reviews
- Implement CI/CD pipelines for automated testing and deployment
- Use proper environment variable management for different stages

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JordiNodeJS) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
