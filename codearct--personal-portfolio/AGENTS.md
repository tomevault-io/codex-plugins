# Copilot Instructions

> Daha fazla bağlam için:
> - [`.github/project-brief.md`](./project-brief.md) — proje kapsamı ve kararlar
> - [`.github/architecture.md`](./architecture.md) — mimari kararlar ve klasör yapısı
> - [`.github/main-instruction.md`](./main-instruction.md) — sabit kurallar (animasyon, import sırası, styling)
> - [`.github/tasks/`](./tasks/) — aktif ve tamamlanmış task'lar (`spec.md`, `instruction.md`, `NOTES.md`)

## Commands

```bash
npm run dev        # Start dev server (Vite)
npm run build      # Type-check + build (tsc -b && vite build)
npm run lint       # ESLint
npm run preview    # Preview production build
```

There are no tests in this project.

## Architecture

Single-page React portfolio. All sections are rendered sequentially in `src/App.tsx`. The page uses:

- **Lenis** for smooth scrolling (initialized in `App.tsx` via `useEffect`)
- **`motion/react`** (the `motion` package) for scroll-triggered animations — NOT `framer-motion`
- **shadcn/ui** (new-york style) for UI primitives in `src/components/ui/`
- **Tailwind CSS v4** with CSS variables for theming, configured via `@import "tailwindcss"` in `src/index.css`

### Data flow

All site content (projects, experience, education, tools, services, stats, nav links) lives in `src/constants/index.tsx`. Types for all data structures are in `src/types/index.ts`. Components consume data from constants — there is no API or state management.

### Sections

Sections map to components: `Hero`, `Stats`, `About`, `Resume`, `Projects`, `Services`. `Testimonials` and `Contact` components exist but are **commented out** in `App.tsx` and `src/constants/index.tsx`.

The `FloatingMenu` and `Sidebar` components are layout/nav — not sections.

## Key Conventions

### Path alias

`@/` resolves to `src/`. Always use `@/` for internal imports (e.g., `@/components/Hero`, `@/lib/utils`).

### Animation pattern

Every section component follows this exact pattern:

```tsx
<motion.section
  initial='hidden'
  whileInView='visible'
  viewport={{ once: true, amount: 0.3 }}
  variants={staggerContainer(0)}  // pass delay in seconds
  id='section-id'
>
  <motion.div variants={fadeUp}>...</motion.div>
</motion.section>
```

`staggerContainer` and `fadeUp` are the only animation variants — both exported from `src/lib/animation.ts`.

### Import grouping

Each file organizes imports into labeled comment blocks in this order:

```ts
/** Node modules */
/** Custom modules */
/** Components */
/** Constants */
/** Assets */
/** Types */
```

### Copyright header

Every source file starts with:

```ts
/**
 * @copyright 2025 Mehmet Akif YILDIZ
 * @license Apache-2.0
 */
```

### Styling

- Use `cn()` from `@/lib/utils` for conditional class merging (wraps `clsx` + `tailwind-merge`)
- Tailwind v4 — no `tailwind.config.js`; theme tokens are CSS variables in `src/index.css`
- `text-primary` and `text-neutral-*` are the primary color tokens used across components

### shadcn/ui

Components are added via the shadcn CLI. The config is in `components.json` (style: `new-york`, icon library: `lucide`). Do not hand-edit files in `src/components/ui/` unless necessary.

### Prettier

Single quotes (`'`), JSX single quotes, semicolons, 2-space indent, `singleAttributePerLine: true` for JSX attributes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/codearct) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
