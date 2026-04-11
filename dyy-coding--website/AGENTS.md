# BookVerse - GitHub Copilot Instructions

This is a modern Gen Z book reading platform built with Next.js 16, featuring a glassmorphism design system and dark mode-first approach.

## Project Overview

BookVerse is a book reading platform designed for Gen Z users, featuring:

- **Modern Design**: Glassmorphism effects, dark mode first, soft gradients
- **Responsive Layout**: Desktop, tablet, and mobile optimized
- **Next.js 16**: Latest Next.js with App Router
- **Tailwind CSS v4**: Custom design system with CSS variables
- **TypeScript**: Full type safety

## Architecture

### Tech Stack
- **Framework**: Next.js 16.1.6 with App Router
- **Styling**: Tailwind CSS v4 with custom design system
- **Icons**: Lucide React
- **Font**: Poppins (Gen Z focused typography)
- **Language**: TypeScript

### Key Directories
```
src/
├── app/                 # Next.js App Router pages
├── components/          # Reusable React components
│   └── ui/             # UI-specific components
public/                 # Static assets
```

## Design System

### Colors (Dark Mode First)
```css
--background: #0F0F14           /* Deep navy base */
--accent-purple: #8B5CF6        /* Primary accent */
--accent-blue: #3B82F6          /* Secondary accent */
--accent-pink: #EC4899          /* Tertiary accent */
--success-green: #10B981        /* Success states */
```

### Key CSS Classes
- `.glass` - Glassmorphism effect with backdrop blur
- `.glow-button` - Gradient buttons with glow effects
- `.gradient-text` - Multi-color gradient text

### Component Patterns
- All interactive components use hover scale and glow effects
- Consistent border radius (1rem to 3rem)
- 8px grid spacing system
- Smooth transitions (200-300ms)

## Development Guidelines

### Component Structure
1. Use 'use client' directive for interactive components
2. Include proper TypeScript interfaces
3. Follow glassmorphism design patterns
4. Implement responsive design

### Styling Conventions
- Use Tailwind classes primarily
- Custom CSS in globals.css for design system
- Consistent hover effects and transitions
- Mobile-first responsive approach

### Code Quality
- TypeScript for all components
- Props interfaces for reusability
- Consistent naming conventions
- Clean, readable component structure

## Pages Overview

### Landing Page (`/`)
- Hero section with animated background
- Trending books carousel
- Category exploration
- Full-width responsive layout

### Browse Books (`/browse`)
- Filter sidebar with genres and ratings
- Grid layout for book cards
- Search functionality
- Infinite scroll or pagination

### Book Detail (`/books/[id]`)
- Split layout: cover + details
- User reviews and ratings
- Related books suggestions
- Social sharing features

### Authentication (`/auth/*`)
- Modern glassmorphism forms
- Social login options
- Form validation and UX

### Search (`/search`)
- Large search input
- Trending and recent searches
- Category quick filters
- Search results with filters

## Deployment

- **Platform**: Vercel (optimized)
- **Build**: `npm run build`
- **Dev**: `npm run dev` (runs on localhost:3000)

The project includes vercel.json for optimal deployment configuration.

## Key Features to Maintain

When making changes or additions:

1. **Preserve Design System**: Maintain glassmorphism aesthetic
2. **Keep Responsive**: Test on mobile, tablet, desktop
3. **Maintain Performance**: Optimize images, lazy load content
4. **TypeScript**: All new code should be properly typed
5. **Accessibility**: Include proper ARIA labels and keyboard navigation

## Component Examples

### BookCard
Central component for displaying books with:
- Cover image with hover effects
- Rating display with stars
- Genre tags
- Read/Like actions
- Glassmorphism styling

### Navigation
Responsive header with:
- Logo with gradient text
- Search functionality
- Mobile hamburger menu
- Authentication state

## Future Considerations

- Add book reader component for immersive reading
- Implement user dashboard and library
- Add community features (reviews, discussions)
- Progressive Web App capabilities
- Advanced search and recommendation engine

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Dyy-Coding)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Dyy-Coding)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
