# Copilot Instructions - Portfolio Landing Page

## Project Overview
Single-page portfolio website for Idris Mohamed showcasing web design, development, and cybersecurity expertise. Built with vanilla HTML, CSS, and JavaScript - no frameworks or build tools.

## Architecture & Structure
- **`index.html`**: Complete single-page structure with semantic sections (hero, about, portfolio, pricing, faq, privacy, footer)
- **`css/styles.css`**: Organized with clear section comments (e.g., `/* === HERO SECTION === */`)
- **`js/main.js`**: Minimal vanilla JS for interactive components only (currently FAQ accordion)

## Design Philosophy & Conventions

### CSS Organization
- **Section-based structure**: Each major section has its own comment block in styles.css
- **Mobile-first responsive**: Uses `clamp()` for fluid typography (e.g., `clamp(2.5rem, 6vw, 4rem)`)
- **Breakpoint**: Single mobile breakpoint at `768px` max-width
- **Color palette**: Minimal monochrome (#111 black, #fff white, grays for text/backgrounds)
- **Grid system**: `repeat(auto-fit, minmax(300px, 1fr))` for strengths cards with 30px gaps

### Component Patterns
- **Cards**: White background, `border-radius: 8px`, `border: 1px solid #eee`, 30px padding
- **Buttons**: 
  - Primary: black background (`#111`), white text, `border-radius: 4px`
  - Hover: slightly lighter (`#333` or `#eee` for inverted)
  - Padding: 16px 48px for CTAs, 14px 36px for secondary buttons
- **Typography**: System font stack, `-0.02em` letter-spacing on large headings

### JavaScript Conventions
- **Vanilla JS only**: No jQuery, React, or other libraries
- **DOMContentLoaded**: Wrap all initialization in `DOMContentLoaded` event
- **Function naming**: Descriptive camelCase (e.g., `initializeFAQ()`, `toggleFAQ()`)
- **Toggle pattern**: Close all items, then open clicked item (unless already open)

### Interactive Components
- **FAQ accordion** (`main.js`): 
  - Uses `.active` class on both question and answer
  - `max-height` transition from 0 to 300px for smooth animation
  - "+" rotates 45deg to "×" when active
  - Only one FAQ open at a time

## Key Files & Responsibilities
- **`index.html`**: Content structure, meta tags (SEO), semantic HTML5 sections
- **`css/styles.css`**: All styling including responsive design and transitions
- **`js/main.js`**: Interactive behavior (accordion, future animations)

## Development Workflow
- **No build process**: Direct file editing, open `index.html` in browser
- **Testing**: Manual testing at 768px breakpoint and desktop widths
- **Deployment**: Static hosting (designed for platforms like Netlify, Vercel, GitHub Pages)

## Content & SEO Specifics
- **Target audience**: Small business owners and startups needing web design services
- **Key differentiators**: E-commerce background, cybersecurity expertise (Security+ cert), UI/UX skills
- **Pricing model**: $800 starting for landing pages, $2,000+ for multi-page sites
- **CTA**: "Book a Call" with Calendly integration (anchors to `#calendly`)

## When Making Changes
- **Adding sections**: Follow existing pattern - section tag → container div → content
- **New interactive features**: Add to `main.js` with initialization in `DOMContentLoaded`
- **Styling new components**: Use established card/button patterns and color palette
- **Responsive design**: Test at mobile breakpoint; leverage `clamp()` for fluid sizing
- **Maintain simplicity**: Avoid dependencies, frameworks, or complex animations (client wants "smooth, simple animations")

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Idurusmohamed)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Idurusmohamed)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
