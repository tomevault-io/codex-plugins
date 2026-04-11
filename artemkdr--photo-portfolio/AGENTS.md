# Copilot Instructions for Photo Portfolio

## Project Overview

This is a modern photo portfolio application built with Next.js 15+ (App Router), TypeScript, and Tailwind CSS. The application displays a photo gallery with modal navigation and supports multiple data storage backends.

## Tech Stack

- **Framework**: Next.js (App Router)
- **Language**: TypeScript
- **Styling**: Tailwind CSS
- **Testing**: Vitest with React Testing Library
- **Image Handling**: Next.js Image optimization
- **Deployment**: Docker support included

## Project Structure

### Core Directories

- **`src/app/`**: Next.js App Router pages and layouts
    - `@modal/`: Parallel route for modal photo display
    - `photo/[name]/`: Individual photo pages
    - Uses intercepting routes `(.)photo/[name]` for modal navigation
- **`src/features/`**: Feature-based architecture
    - `photo-wall/`: Main gallery functionality with thumbnails, full images, and modal components
    - `theme-switcher/`: Dark/light theme toggle functionality
- **`src/data/`**: Data layer with pluggable storage adapters
    - `blob-storage/`: Azure Blob Storage integration
    - `file-system/`: Local file system storage
    - `google-photos/`: Google Photos API integration
    - `dummy/`: Mock data for testing
- **`src/content/`**: Content configuration and metadata
- **`public/photos/`**: Static photo assets

### Key Files

- `app.config.ts`: Application-wide configuration
- `content.json`: Photo metadata and content
- `next.config.ts`: Next.js configuration
- `tailwind.config.ts`: Tailwind CSS customization

## Code Conventions

### TypeScript

- Strict mode enabled
- Prefer type inference where possible
- Use interfaces for objects, types for unions/intersections
- All components should be strongly typed

### React/Next.js

- Use Server Components by default
- Add `'use client'` only when necessary (interactivity, hooks, context)
- Prefer function components over class components
- Use the `Photo` type from `src/features/photo-wall/types/photo.ts`

### Styling

- Use Tailwind CSS utility classes
- Follow responsive design patterns (mobile-first)
- Maintain dark mode support via theme provider

### Testing

- Write tests alongside implementation files (`.test.ts`, `.test.tsx`)
- Use Vitest for unit tests
- Test utilities, components, and data layers
- Run tests with `npm run test`

## Data Layer Pattern

The application uses a pluggable data provider pattern:

1. Define photo data source in configuration
2. Data providers implement a consistent interface
3. Photos are fetched and transformed to the `Photo` type
4. Supports local files, cloud storage, and external APIs

## Feature Patterns

### Photo Wall

- Thumbnail grid with lazy loading
- Modal overlay for full-size viewing
- Keyboard navigation (arrow keys, Escape)
- Context providers for photos, direction, and window size
- Utility functions for throttling and debouncing

### Theme Switching

- Context-based theme management
- Persists user preference
- Supports system preference detection

## Important Considerations

### Performance

- Images are optimized via Next.js Image component
- Implement proper lazy loading for thumbnails
- Preload adjacent images in modal view for smooth navigation
- Use throttling/debouncing for scroll and resize events

### Navigation

- Modal navigation uses Next.js intercepting routes
- Direct links to photos work independently of modal state
- Back button should close modal and return to gallery

### Accessibility

- Images should have proper alt text
- Keyboard navigation must work throughout
- Focus management in modals
- Theme switcher should be accessible

## Development Workflow

1. Install dependencies: `npm install`
2. Run development server: `npm run dev`
3. Run tests: `npm run test`
4. Build for production: `npm run build`
5. Docker build: Use provided Dockerfile

## When Making Changes

### Adding Photos

- Place images in `public/photos/`
- Update `src/content/content.json` with metadata
- Ensure proper naming conventions

### Adding Features

- Create new feature folder under `src/features/`
- Include components, contexts, types, and utils as needed
- Write corresponding tests
- Update configuration if needed

### Modifying Data Sources

- Implement the data provider interface
- Add tests in `data.test.ts`
- Update app configuration to enable the new source

### Styling Changes

- Use existing Tailwind classes when possible
- Extend `tailwind.config.ts` for custom values
- Maintain dark mode compatibility
- Test responsive behavior at all breakpoints

## Common Tasks

### Adding a New Photo

1. Add image to `public/photos/`
2. Add entry to `content.json` with name, title, description, date
3. Test that it appears in the gallery

### Creating a New Component

1. Create component file in appropriate feature folder
2. Create corresponding test file
3. Export from feature if needed externally
4. Add TypeScript types/interfaces

### Debugging

- Check browser console for client-side errors
- Review Next.js build output for SSR issues
- Use React DevTools for component inspection
- Verify image paths and optimization

## Best Practices

- Keep components small and focused
- Separate concerns (UI, logic, data)
- Write tests for new functionality
- Use semantic HTML
- Optimize images before adding to public folder
- Follow existing patterns and conventions
- Document complex logic with comments
- Prefer composition over inheritance

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/artemkdr)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/artemkdr)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
