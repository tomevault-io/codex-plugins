# AI Coding Instructions for guidebareme

## Project Overview
This is a static Next.js website displaying the French "GUIDE-BARÈME POUR L'ÉVALUATION DES DÉFICIENCES ET INCAPACITÉS DES PERSONNES HANDICAPÉES" (Disability Evaluation Guide). The site was migrated from HTML content and uses Next.js App Router for static export.

## Architecture
- **Static Site**: Configured with `output: 'export'` in `next.config.mjs` for static generation.
- **Content Structure**: Main content is stored as an HTML string in `app/content.tsx` and rendered via `dangerouslySetInnerHTML` in `app/page.tsx`.
- **Styling**: Tailwind CSS with `@tailwindcss/typography` plugin for prose styling (e.g., `prose lg:prose-xl prose-stone` classes).
- **Layout**: Standard Next.js layout in `app/layout.tsx` with Inter font.

## Key Patterns
- **Content Import**: Import HTML content as a string from `content.tsx` and render directly in components.
- **No Dynamic Data**: Pure static content; no API calls or database interactions.
- **Typography Classes**: Use Tailwind prose utilities for readable text formatting, e.g., `prose-stone max-w-none` for full-width styled content.

## Development Workflow
- **Start Dev Server**: `npm run dev` - Runs on localhost:3000.
- **Build Static Site**: `npm run build` - Generates static files in `out/` directory.
- **Linting**: `npm run lint` - Uses ESLint for code quality.
- **No Tests**: Project lacks test scripts; focus on manual verification.

## Conventions
- **File Structure**: Keep content in `app/content.tsx` as exported HTML string; render in `app/page.tsx`.
- **Styling**: Prefer Tailwind utilities; use prose for long-form content.
- **Static Export**: Ensure all assets are statically accessible; no server-side rendering.

## Key Files
- `app/page.tsx`: Main page component importing and rendering content.
- `app/content.tsx`: Contains the full HTML content as a string export.
- `app/layout.tsx`: Root layout with metadata and font setup.
- `next.config.mjs`: Static export configuration.

When modifying content, update the HTML string in `content.tsx` directly. For styling changes, adjust Tailwind classes in `page.tsx` or `globals.css`. Always build and verify static output after changes.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adamico)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/adamico)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
