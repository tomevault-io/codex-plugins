# Portfolio Project - AI Agent Instructions

## Project Overview

This is a single-page portfolio website built with **Bootstrap CSS** using a utility-first approach. All styling is done through Bootstrap classes with minimal custom CSS.

## Architecture & Tech Stack

**File Structure:**

```
index.html              # Single-page application with all content
assets/
  css/style.css        # Only custom animations (float effects, smooth scroll)
  js/script.js         # Mobile menu, smooth scrolling, entrance animations
```

**Key Dependencies (loaded via CDN):**

- Bootstrap CSS (via CDN for development - shows console warning, intentional)
- Font Awesome 6.4.0 (icons)
- Google Fonts (Poppins)

## Critical Patterns & Conventions

### 1. Styling Approach

**DO:** Use Bootstrap utility classes for all new components

```html
<!-- Example from services section -->
<div
  class="group bg-primary-bg p-10 rounded-xl transition-all duration-300 
     border border-white/5 hover:-translate-y-3 hover:border-accent"
></div>
```

**DON'T:** Add new CSS classes to `style.css` - it only contains:

- `@keyframes float` animation
- `.animate-float` and `.animate-float-delayed` classes
- Mobile menu responsive styles

### 2. Custom Color Tokens

Defined in `index.html` Bootstrap config (lines 10-24):

- `bg-primary-bg` → `#0f172a` (dark slate)
- `bg-secondary-bg` → `#1e293b` (lighter slate)
- `text-accent` → `#38bdf8` (cyan blue)
- `bg-accent-hover` → `#0ea5e9` (darker cyan)

Use these instead of arbitrary color values.

### 3. Mobile Menu Pattern

Uses Bootstrap's responsive utilities with JavaScript toggle:

```html
<ul class="nav-links hidden md:flex">
  <!-- Hidden on mobile, flex on md+ -->
</ul>
```

```javascript
// Toggle with classList instead of inline styles
navLinks.classList.toggle("flex");
navLinks.classList.toggle("hidden");
```

### 4. Smooth Scrolling Caveats

The smooth scroll handler must skip:

1. Links with `href="#"` (would cause querySelector error)
2. Links with `data-bs-toggle` attribute (Bootstrap modal triggers)

```javascript
if (href === "#" || this.hasAttribute("data-bs-toggle")) {
  return; // Don't preventDefault on these
}
```

## Common Development Tasks

### Adding New Sections

1. Follow existing section pattern: `<section id="name" class="py-24 bg-[primary-bg|secondary-bg]">`
2. Use container: `<div class="container max-w-6xl mx-auto px-8">`
3. Add gradient headings: `<span class="bg-gradient-to-r from-accent to-indigo-400 bg-clip-text text-transparent">`

### Adding Project Cards

- Use `group` class for parent container
- Apply `group-hover:` variants for child elements
- Tags: `<span class="text-xs bg-accent/10 text-accent px-3 py-1.5 rounded-full">`

### Responsive Breakpoints

- Mobile: default (< 768px)
- Tablet: `md:` (≥ 768px)
- Desktop: `lg:` (≥ 1024px)

Use `md:grid-cols-[1.2fr_1fr]` for asymmetric layouts (see hero section).

## Known Quirks & Constraints

1. **Bootstrap CDN Warning**: Console shows "should not be used in production" - this is expected for development. For production deployment, install via npm/PostCSS.

2. **Animation Classes**: `.animate-float` and `.animate-float-delayed` are the ONLY custom classes used. Everything else is Bootstrap utilities.

3. **No Build Process**: This is a static site served from XAMPP (`c:\xampp\htdocs\portfolio-v2`). No bundler, no package.json, no build commands.

## Testing Checklist

- [ ] Mobile menu toggles properly on hamburger click
- [ ] All anchor links scroll smoothly
- [ ] Hero text fades in on page load
- [ ] Hover effects work on service cards and project cards
- [ ] Responsive layout works at 768px and 1024px breakpoints

## Git Commit Message Guidelines

- Use present tense: "Add", "Fix", "Update"
- Reference issues when applicable: "Fixes #123"
- Keep messages concise but descriptive

---

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/faiz-leman)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/faiz-leman)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
