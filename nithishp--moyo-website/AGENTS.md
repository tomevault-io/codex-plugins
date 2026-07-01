# MOYO Website — Copilot Instructions

E-commerce/marketing site for MOYO consumer products (tissue boxes, foil, bags). Built with Next.js 15 App Router, React 19, TypeScript 5, and Tailwind CSS 4.

## Build & Dev

```bash
npm run dev      # Start dev server (localhost:3000)
npm run build    # Production build
npm run lint     # ESLint
```

No test framework is configured.

## Architecture

| Area                  | Location                                                    | Notes                                                        |
| --------------------- | ----------------------------------------------------------- | ------------------------------------------------------------ |
| Landing page          | `app/page.tsx`                                              | Assembles section components                                 |
| Root layout           | `app/layout.tsx`                                            | Wraps MainNav + Sonner toast                                 |
| Product catalog       | `data/index.js`                                             | Static `ProductData` array — filter/find client-side         |
| Product routes        | `app/product-categories/[id]/`, `app/product-details/[id]/` | Dynamic routes using `params.id`                             |
| Feedback (MongoDB)    | `lib/db/database.ts`, `lib/models/feedback.ts`              | Uses `connectToDB()` + Mongoose; needs `MONGODB_URI` env var |
| Base UI               | `components/ui/`                                            | shadcn/ui-style components (Button, Card, Dialog, …)         |
| Component library     | `components/inatorscomponents/`                             | 25+ category folders of reusable Tailwind components         |
| Page-level components | `app/_components/`                                          | Hero, Stats, Features, ProductSection, Navbar, etc.          |

No API route handlers exist. Email is sent client-side via EmailJS.

## Conventions

**File extensions:**

- `app/_components/` uses `.jsx` — page-specific, client-side components
- `components/ui/` and `components/inatorscomponents/` use `.tsx` — typed library components
- New files in `components/` should be `.tsx`; new files in `app/_components/` can be `.jsx`

**Styling:**

- Tailwind-first. Always use `cn()` from `lib/utils.ts` for conditional/merged classes.
- Use CVA (`class-variance-authority`) for multi-variant components (see `components/ui/button.tsx` as reference).
- Dark mode is class-based (`dark:`).
- CSS Modules are limited to `SplashScreen.module.css` — don't add new ones.

**Components:**

- All page-level components use `'use client'` at the top.
- Reusable components in `components/inatorscomponents/` should be pure (no internal data fetching).
- Animation: Framer Motion (`motion.*` components). Magic effects in `components/magicui/` or `components/animata/`.
- Product images live in `public/products/`; brand assets in `public/brand/`.

**Data:**

- Product data is sourced from `data/index.js` → `ProductData`. Import as `import { ProductData } from '@/data'`.
- MongoDB is used only for the feedback model. Run `connectToDB()` before any Mongoose operation.

## Common Pitfalls

- `tailwind.config.js` **and** `tailwind.config.ts` both exist — edits go in `tailwind.config.ts` (TypeScript version takes precedence in this setup).
- There is no `pages/` directory — this is App Router only.
- `next/image` is required for all images (not `<img>` tags).

---
> Source: [nithishp/moyo-website](https://github.com/nithishp/moyo-website) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-30 -->
