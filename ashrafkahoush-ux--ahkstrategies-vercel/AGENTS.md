# AHKStrategies Development Guide

## Project Architecture

This is a **bilingual Next.js 14** corporate website with **deals-first** lead generation focus, deployed on Vercel.

### Key Architectural Decisions

- **Nested app directory structure**: Root dir is `/ahkstrategies/ahkstrategies/` (not project root)
- **Internationalization**: Uses `next-intl` with `[locale]` dynamic routing for English/Arabic
- **Form-to-CRM integration**: Direct Zoho CRM API integration for lead/partner capture
- **Client-side analytics**: Lazy-loaded Plausible + GA4 to avoid SSR issues

## Critical File Patterns

### Internationalization Setup
- **Middleware**: `src/middleware.ts` handles locale routing with `localePrefix: 'as-needed'`
- **Nested layouts**: Root layout + locale-specific layout in `app/[locale]/layout.tsx`
- **Translation files**: `src/messages/{en,ar}.json` - always update both when adding strings
- **Component i18n**: Use `useTranslations('ComponentName')` hook pattern

### API Routes & CRM Integration
- **Zoho integration**: `src/lib/zoho.ts` exports `createLead()` and `createPartner()`
- **Environment variables**: Zoho config requires `ZOHO_BASE_URL`, `ZOHO_ACCESS_TOKEN`
- **Field mapping**: API routes in `app/[locale]/api/` map form data to Zoho schema
- **Error handling**: Always wrap Zoho calls in try/catch with proper status codes

### Component Conventions
- **Client components**: Mark with `"use client"` for forms, analytics, interactive elements
- **Form state**: Use `useState<'idle' | 'loading' | 'success' | 'error'>` pattern
- **Locale-aware fetching**: Forms submit to `/${locale}/api/endpoint` paths
- **Styling**: Custom Tailwind theme with primary (navy) and accent (gold) brand colors

## Development Workflow

### Environment Setup
```bash
cd ahkstrategies/ahkstrategies  # Navigate to actual project root
npm run dev                     # Development server
```

### Key Scripts
- `npm run build` - Production build (required before deployment)
- `npm run lint` - Next.js ESLint rules

### Analytics Configuration
- **Client-only loading**: Analytics component uses `dynamic` import with `ssr: false`
- **Environment variables**: Use `NEXT_PUBLIC_` prefix for client-side analytics IDs
- **Dual tracking**: Supports both Plausible and Google Analytics simultaneously

## Common Patterns

### Adding New Pages
1. Create in `src/app/[locale]/` directory
2. Update both `src/messages/en.json` and `src/messages/ar.json`
3. Follow component pattern: default export function, use `useTranslations()`

### Form Implementation
- Map HTML form fields to Zoho CRM field names in API route
- Use semantic HTML names (`Name`, `Email`, `Phone`, `Company`, `Description`)
- Handle loading states and user feedback consistently

### Styling Guidelines
- Use custom theme colors: `bg-primary`, `text-accent`, etc.
- Responsive design with `md:` breakpoints for desktop layouts
- Component-scoped CSS classes, avoid global styles outside `globals.css`

## External Dependencies

- **CRM**: Zoho CRM API (requires OAuth token management)
- **Analytics**: Plausible.io + Google Analytics 4
- **Deployment**: Vercel (automatic from main branch)
- **Styling**: Tailwind CSS v4.1.x with PostCSS

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ashrafkahoush-ux)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/ashrafkahoush-ux)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
