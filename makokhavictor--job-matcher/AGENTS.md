# GitHub Copilot Instructions for Next.js Project

This document provides instructions and best practices for GitHub Copilot when working with this Next.js project.

## Project Structure

- Follow the App Router file structure convention:
  - `app/` directory for routes and layouts
  - `components/` for reusable UI components
  - `lib/` for utility functions and shared code
  - `public/` for static assets
  - `styles/` for global CSS and styling

## Color Theme

This job matching with CV platform uses the following professional color theme:

- **Primary**: `#2563EB` (Royal Blue) - Used for primary buttons, main navigation elements, and branded areas
- **Secondary**: `#475569` (Slate Gray) - Used for secondary UI elements, subheadings, and supporting content
- **Accent**: `#10B981` (Emerald Green) - Used for accents, success indicators, calls-to-action, and highlighting matches
- **Neutral Background**: `#F8FAFC` (Light Gray) - Used for page backgrounds and cards
- **Text Primary**: `#1E293B` (Dark Slate) - Main text color
- **Text Secondary**: `#64748B` (Medium Gray) - Secondary text and less important information

This color scheme conveys professionalism, trust, and success - perfect for a job matching platform. The blue represents reliability and professionalism, while the green accent indicates success and growth, appropriate for highlighting matches and positive outcomes.

## Code Style & Best Practices

### General

- Use TypeScript for all new files
- Prefer functional components with hooks over class components
- Use named exports instead of default exports where appropriate
- Keep components small and focused on a single responsibility
- Write self-documenting code with descriptive variable/function names
- Use absolute imports instead of relative path hell (e.g., `@/components/Button` instead of `../../../components/Button`)
- Use shadcn for the UI components

### React & Next.js

- Use Server Components by default unless client interactivity is needed
- Add "use client" directive only to components that:
  - Use React hooks
  - Need browser APIs
  - Require event listeners
  - Manage client-side state
- Use the `Link` component for internal navigation, not `<a>` tags
- Implement proper loading states and error boundaries
- Use Next.js Image component for optimized images
- Implement proper metadata for SEO using metadata objects or generateMetadata

### Data Fetching

- Prefer server-side data fetching in Server Components
- Use React Query for client-side data fetching when needed
- Implement proper loading, error, and empty states
- Use revalidation strategies appropriately (ISR with revalidate, on-demand revalidation)

### State Management

- Use React Context API for global state when needed
- Prefer local component state with useState/useReducer for component-specific state
- Consider Zustand for more complex state management needs

### Styling

- Use CSS Modules or Tailwind CSS for component styling
- Follow mobile-first responsive design approach
- Maintain a consistent design system with reusable variables/themes
- Implement the color theme specified above throughout the application
- Use the color theme variables in a Tailwind config or CSS variables

### Forms

- Use controlled components for form inputs when appropriate
- Implement proper form validation with helpful error messages
- Use React Hook Form for complex forms

### Performance

- Implement code splitting using dynamic imports where beneficial
- Optimize and lazy-load images
- Use Next.js font optimization
- Memoize expensive computations with useMemo and useCallback when appropriate
- Virtualize long lists with react-window or similar libraries

### Accessibility

- Use semantic HTML elements
- Ensure proper keyboard navigation
- Add appropriate ARIA attributes when necessary
- Maintain sufficient color contrast with the chosen color theme
- Support screen readers with proper labels and descriptions

### Testing

- Write unit tests for utility functions
- Write component tests for UI components
- Add end-to-end tests for critical user flows
- Implement proper test coverage

### Error Handling

- Use try/catch blocks for error handling
- Create custom error handling components
- Implement proper error logging

## Common Patterns to Follow

### API Routes

```typescript
// Example API route pattern with proper error handling
export async function GET(request: Request) {
  try {
    // Logic here
    return Response.json({ data: result })
  } catch (error) {
    console.error('Failed to process request:', error)
    return Response.json(
      { error: 'Something went wrong' },
      { status: 500 }
    )
  }
}
```

### Server Component

```typescript
// Server component pattern
import { getItems } from '@/lib/data'

export default async function ItemsList() {
  // Data fetching directly in server component
  const items = await getItems()
  
  return (
    <ul className="grid gap-4">
      {items.map((item) => (
        <li key={item.id}>{item.name}</li>
      ))}
    </ul>
  )
}
```

### Client Component

```typescript
'use client'

import { useState } from 'react'

export default function Counter() {
  const [count, setCount] = useState(0)
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  )
}
```

### Data Fetching with React Query

```typescript
'use client'

import { useQuery } from '@tanstack/react-query'

export default function UserProfile({ userId }) {
  const { data, error, isLoading } = useQuery({
    queryKey: ['user', userId],
    queryFn: () => fetch(`/api/users/${userId}`).then(res => res.json())
  })
  
  if (isLoading) return <div>Loading...</div>
  if (error) return <div>Error loading user data</div>
  
  return (
    <div>
      <h1>{data.name}</h1>
      <p>{data.email}</p>
    </div>
  )
}
```


## Avoid These Anti-patterns

- Don't mix client and server code - respect the "use client" boundary
- Don't use document or window in Server Components
- Don't fetch data in Client Components when it can be done in Server Components
- Don't use Redux for simple state management cases
- Don't use excessive prop drilling - use context or composition instead
- Don't create large monolithic components
- Don't ignore TypeScript errors/warnings
- Don't use CSS-in-JS libraries with Server Components
- Don't ignore accessibility concerns

## Testing Guidelines

- Use React Testing Library for component tests
- Use Jest for unit tests
- Use Playwright or Cypress for end-to-end tests
- Follow the Testing Trophy approach (more integration tests, fewer unit tests)

## Performance Optimization Guidelines

- Implement proper code splitting
- Use next/image for image optimization
- Use next/font for font optimization
- Implement prefetching for anticipated routes
- Use Suspense boundaries for improved loading states
- Profile and optimize component rendering

## SEO Best Practices

- Implement proper metadata for each page
- Use semantic HTML
- Implement proper structured data where relevant
- Ensure mobile-friendliness
- Optimize page load speed

## Deployment Considerations

- Use environment variables for configuration
- Implement proper error monitoring
- Set up proper logging
- Consider using Edge functions for global performance

## Tailwind CSS v4 Configuration for Theme

Add this configuration to your tailwind.config.js to implement the project's color theme using Tailwind CSS v4 syntax:

```javascript
export default {
  theme: {
    // Tailwind v4 uses the extend pattern differently
    colors: {
      // Base colors that won't be overridden
      white: '#FFFFFF',
      black: '#000000',
      transparent: 'transparent',
      // Theme colors
      primary: {
        DEFAULT: '#2563EB',
        50: '#EBF2FF',
        100: '#DBEAFE',
        200: '#BFDBFE',
        300: '#93C5FD',
        400: '#60A5FA',
        500: '#3B82F6',
        600: '#2563EB',
        700: '#1D4ED8',
        800: '#1E40AF',
        900: '#1E3A8A',
        950: '#172554',
      },
      secondary: {
        DEFAULT: '#475569',
        50: '#F8FAFC',
        100: '#F1F5F9',
        200: '#E2E8F0',
        300: '#CBD5E1',
        400: '#94A3B8',
        500: '#64748B',
        600: '#475569',
        700: '#334155',
        800: '#1E293B',
        900: '#0F172A',
        950: '#020617',
      },
      accent: {
        DEFAULT: '#10B981',
        50: '#ECFDF5',
        100: '#D1FAE5',
        200: '#A7F3D0',
        300: '#6EE7B7',
        400: '#34D399',
        500: '#10B981',
        600: '#059669',
        700: '#047857',
        800: '#065F46',
        900: '#064E3B',
        950: '#022C22',
      },
      // Re-create gray scale (used for backgrounds, borders, etc.)
      gray: {
        50: '#F9FAFB',
        100: '#F3F4F6',
        200: '#E5E7EB',
        300: '#D1D5DB',
        400: '#9CA3AF',
        500: '#6B7280',
        600: '#4B5563',
        700: '#374151',
        800: '#1F2937',
        900: '#111827',
        950: '#030712',
      },
    },
  },
  // Enable the Tailwind v4 JIT compiler
  future: {
    hoverOnlyWhenSupported: true,
  },
  // Use Tailwind v4 content source configuration
  content: [
    './app/**/*.{js,ts,jsx,tsx}',
    './components/**/*.{js,ts,jsx,tsx}',
  ],
  // For dark mode with Tailwind v4
  darkMode: 'class',
}
```

### CSS Variables Setup (Alternative Approach)

For more flexibility, consider using CSS variables with Tailwind v4:

```css
/* globals.css */
:root {
  /* Primary (Blue) */
  --color-primary: 37, 99, 235; /* #2563EB */
  --color-primary-50: 235, 242, 255; /* #EBF2FF */
  --color-primary-100: 219, 234, 254; /* #DBEAFE */
  --color-primary-500: 59, 130, 246; /* #3B82F6 */
  --color-primary-900: 30, 58, 138; /* #1E3A8A */
  
  /* Secondary (Slate) */
  --color-secondary: 71, 85, 105; /* #475569 */
  --color-secondary-50: 248, 250, 252; /* #F8FAFC */
  --color-secondary-500: 100, 116, 139; /* #64748B */
  --color-secondary-900: 15, 23, 42; /* #0F172A */
  
  /* Accent (Green) */
  --color-accent: 16, 185, 129; /* #10B981 */
  --color-accent-50: 236, 253, 245; /* #ECFDF5 */
  --color-accent-500: 52, 211, 153; /* #34D399 */
  --color-accent-900: 6, 78, 59; /* #064E3B */
}

/* Dark mode */
.dark {
  /* Darker theme variables */
  --color-primary: 29, 78, 216; /* #1D4ED8 (darker blue) */
  --color-secondary: 51, 65, 85; /* #334155 (darker slate) */
  --color-accent: 5, 150, 105; /* #059669 (darker green) */
}
```

Combine with this Tailwind v4 config:

```javascript
export default {
  theme: {
    extend: {
      colors: {
        primary: {
          DEFAULT: 'rgb(var(--color-primary))',
          50: 'rgb(var(--color-primary-50))',
          100: 'rgb(var(--color-primary-100))',
          500: 'rgb(var(--color-primary-500))',
          900: 'rgb(var(--color-primary-900))',
        },
        secondary: {
          DEFAULT: 'rgb(var(--color-secondary))',
          50: 'rgb(var(--color-secondary-50))',
          500: 'rgb(var(--color-secondary-500))',
          900: 'rgb(var(--color-secondary-900))',
        },
        accent: {
          DEFAULT: 'rgb(var(--color-accent))',
          50: 'rgb(var(--color-accent-50))',
          500: 'rgb(var(--color-accent-500))',
          900: 'rgb(var(--color-accent-900))',
        },
      },
    },
  },
}
```

---
> Source: [makokhavictor/job-matcher](https://github.com/makokhavictor/job-matcher) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-15 -->
