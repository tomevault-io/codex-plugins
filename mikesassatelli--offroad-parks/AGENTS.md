# Offroad Parks App - AI Development Guide

## Project Overview

This is a Next.js application that helps users discover and manage off-road park information. The app showcases parks with details about terrain, amenities, and trail information for UTV/off-road enthusiasts.

## Architecture & Data Flow

- Built using Next.js 16+ with App Router and React Server Components
- UI components use shadcn/ui (based on Radix UI) with Tailwind CSS
- Main component: `src/components/ui/OffroadParksApp.tsx` - handles core app logic
- Data model defined in `src/lib/types.ts` with `Park`, `Amenity`, and `Terrain` types
- Park data stored in `src/data/parks.ts`

## Key Development Patterns

1. **Component Structure**
   - UI components are in `src/components/ui/`
   - Use client-side React with "use client" directive
   - State management via React hooks (useState, useMemo)

2. **Type Safety**
   - Strict TypeScript usage throughout
   - Use defined types from `types.ts` for data consistency
   - Example: `type Terrain = "sand" | "mud" | "hardpack" | ...`

3. **Styling**
   - Tailwind CSS with custom class composition
   - Use shadcn/ui components with consistent styling patterns
   - Example class pattern: `className="text-sm text-gray-600 mt-1"`

## Development Workflow

```bash
# Install dependencies
npm install

# Start development server
npm run dev

# Build for production
npm run build

# Start production server
npm start

# Run linting
npm run lint
```

## Common Tasks

1. **Adding New UI Components**
   - Place in `src/components/ui/`
   - Use shadcn/ui patterns with Radix UI primitives
   - Include TypeScript types and "use client" when needed

2. **Modifying Park Data**
   - Update `src/data/parks.ts`
   - Ensure data matches `Park` type interface
   - Include required fields: id, name, state, utvAllowed, terrain, amenities

3. **Adding New Features**
   - Follow existing patterns in `OffroadParksApp.tsx`
   - Use TypeScript for type safety
   - Leverage existing UI components when possible

## Project Conventions

- Use named exports for components and types
- Follow existing component structure with Cards for park display
- Maintain type safety with TypeScript interfaces
- Use Tailwind for styling with shadcn/ui components

---
> Source: [mikesassatelli/offroad-parks](https://github.com/mikesassatelli/offroad-parks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-22 -->
