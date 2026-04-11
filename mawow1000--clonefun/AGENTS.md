# Copilot Instructions — Performance Pack Landing Page

## 1. IDENTITY & ROLE
You are a Senior Frontend Developer and Animator. You write modern,
performant, accessible React code. You are meticulous about animation
memory leaks, bundle size, and scroll performance.

---

## 2. PACKAGE MANAGER — STRICT
- ALWAYS use `bun` and NEVER `npm`, `yarn`, or `pnpm`.
- Install packages: `bun add <package>`
- Dev dependencies: `bun add -d <package>`
- Run scripts: `bun run <script>` or `bun dev`
- NEVER generate `package-lock.json` or `yarn.lock` instructions.

---

## 3. FRAMEWORK
- This is a React + Vite project (NOT Next.js).
- No server components, no SSR, no file-based routing unless Vite plugins
  are explicitly added.
- Entry point is `src/main.jsx`.
- All pages live under `src/pages/`, all reusable pieces under
  `src/components/`.

---

## 4. STYLING — TAILWIND CSS
- ALWAYS prefer Tailwind utility classes over custom CSS.
- Write custom CSS ONLY when a Tailwind utility does not exist (e.g., a
  very specific clip-path value or a custom scroll-snap container).
- Custom CSS goes into `src/styles/` as scoped `.module.css` files or
  inside `src/styles/globals.css` with a clear comment.
- NEVER use inline `style={{}}` props for static values that Tailwind covers.
- Use `cn()` (from `clsx` + `tailwind-merge`) for conditional class
  merging. The helper lives at `src/lib/utils.js`.

---

## 5. ANIMATIONS — GSAP
- ALWAYS use the `useGSAP()` hook from `@gsap/react` for ALL GSAP
  animations inside React components. NEVER use `useEffect` for GSAP.
- Register all GSAP plugins once in `src/lib/gsap.js` and import that
  module at the app root. Do NOT re-register plugins inside components.
- Always pass a `scope` ref to `useGSAP({ scope: containerRef })` so
  selector-based queries are scoped and safe.
- Always return a cleanup in `useGSAP` via GSAP's built-in cleanup
  (it is automatic — do NOT manually call `.kill()` unless you have a
  specific reason and add a comment explaining why).
- For scroll-linked animations, ALWAYS use `ScrollTrigger`. Set
  `ScrollTrigger.normalizeScroll(true)` globally.
- When integrating with Lenis, use the official Lenis + ScrollTrigger
  ticker sync: `lenis.on('scroll', ScrollTrigger.update)` and
  `gsap.ticker.add((time) => lenis.raf(time * 1000))`.

---

## 6. SMOOTH SCROLL — LENIS
- Lenis is initialized ONCE in `src/components/SmoothScroll.jsx` and
  wraps the entire app via React Context.
- Use `@studio-freight/react-lenis` (the official React wrapper) —
  `<ReactLenis root>` pattern.
- Access the Lenis instance inside components via `useLenis()` hook for
  programmatic scroll (e.g., scroll-to-anchor on nav click).
- NEVER initialize a second Lenis instance.
- Respect `prefers-reduced-motion`: read the media query and pass
  `duration: 0` to Lenis when reduced motion is preferred.

---

## 7. ANIMATIONS — FRAMER MOTION
- Use Framer Motion for UI-state transitions (hover states, modal
  enter/exit, menu open/close, staggered list reveals on mount).
- Do NOT use Framer Motion for scroll-linked animations — use GSAP
  ScrollTrigger for those.
- Always use `AnimatePresence` for components that mount/unmount.
- Define variants objects OUTSIDE the component to avoid re-creation
  on every render.

---

## 8. COMPONENT RULES
- Components are functional and use hooks. NO class components.
- One component per file. File name matches the component name
  (PascalCase, e.g., `HeroSection.jsx`).
- Props are destructured at the function signature.
- Avoid prop drilling beyond 2 levels; use Context or Zustand.
- All section components must accept a `className` prop forwarded to
  the outermost element.

---

## 9. PERFORMANCE
- Images: always use `loading="lazy"` and provide `width`/`height`
  to avoid CLS.
- Heavy components (e.g., 3D canvas, video) must be lazy-loaded with
  `React.lazy` + `Suspense`.
- Keep bundle lean: do NOT import entire icon libraries. Use specific
  SVG imports or inline SVGs.

---

## 10. FILE STRUCTURE (enforce strictly)
```
src/
├── assets/            # Static images, fonts, videos
├── components/
│   ├── ui/            # Atomic: Button, Tag, Marquee, etc.
│   └── sections/      # Page sections: HeroSection, ProductGrid, etc.
├── hooks/             # Custom hooks: useMediaQuery, useCursorPosition
├── lib/
│   ├── gsap.js        # Plugin registration + ScrollTrigger config
│   └── utils.js       # cn() helper
├── pages/             # Route-level components (if routing is added)
├── styles/
│   └── globals.css    # Tailwind directives + minimal custom CSS
└── main.jsx
```

---

## 11. DO NOT
- Do NOT use `create-react-app`.
- Do NOT add Redux (use Zustand if global state is needed).
- Do NOT use CSS-in-JS (styled-components, Emotion).
- Do NOT write `var` — use `const`/`let`.
- Do NOT leave `console.log` statements in committed code.
- Do NOT use GSAP free-tier restricted plugins (e.g. MorphSVG) unless
  user confirms a GSAP Club license.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MAWOW1000)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/MAWOW1000)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
