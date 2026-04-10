# Funnkar Design House – Copilot AI Agent Instructions

## Project Purpose & Structure
Funnkar Design House is a static, dark-themed creative agency portfolio. It showcases branding, product design, graphics, VFX, and animation services. The site is built with vanilla HTML, CSS, and JavaScript—no frameworks or build tools.

**Key files/directories:**
- `index.html` – Landing page (hero, about, service previews, portfolio preview)
- `services.html` – Service details (grid sections, image + text pairs)
- `portfolio.html` – Filterable project gallery (category filter, project cards)
- `project.html` – Project case study template (challenge, approach, results)
- `contact.html` – Contact form (client-side validation) and FAQ accordion
- `image-guide.html` – Interactive image placeholder generator for development
- `css/global.css` – CSS variables, resets, base styles
- `css/components.css` – All reusable UI components (cards, buttons, overlays, forms, menu, grid)
- `css/landing.css` – Landing page hero and animation styles
- `js/main.js` – Global JS: hamburger menu, smooth scroll, nav state, scroll animations
- `js/portfolio.js` – Portfolio filtering logic (category filter, show/hide with animation)
- `js/contact.js` – Contact form validation, FAQ accordion logic
- `assets/images/` – All imagery (see `assets/images/README.md` for required files/specs)
- `README.md` – Complete project documentation
- `QUICKSTART.md` – Quick setup guide
- `PROJECT-COMPLETE.md` – Project completion checklist

## Design & UI Patterns
- **Dark mode:** Uses `--color-bg-primary`/`secondary` and blue/coral accents (see `css/global.css`)
- **Mobile-first:** Responsive breakpoints standardized at **480px (mobile), 768px (tablet), 1024px (desktop)**
- **Responsive Strategy:**
  - Base styles optimized for mobile (320px+)
  - Media queries ADD features for larger screens (not subtract)
  - Use `clamp()` for fluid typography scaling
  - Use `max-width` containers with flexible grid layout
  - Minimum touch targets 44px × 44px on mobile
  - All images use `loading="lazy"` for performance
- **Hero sections:** Full viewport, centered, with animated backgrounds (GIFs/WebP preferred)
  - Mobile (480px): 60-70vh height
  - Tablet (768px): 70-80vh height
  - Desktop (1024px+): 90-100vh height
- **Cards & grids:** `.card`, `.portfolio-item`, `.grid-2`, `.grid-3` for layout
  - Desktop: 3-column grid for portfolio
  - Tablet (768px): 2-column grid
  - Mobile (480px): Single column
- **Hamburger menu:** Full-screen overlay, slide-in, blur backdrop (`js/main.js`)
  - Button: 44px × 44px minimum (top-right)
  - Menu: 100% viewport width/height overlay
  - Close button: 40px × 40px minimum
  - Test on all breakpoints
- **Portfolio filter:** `.filter-btn` toggles, filters `.portfolio-item` by `data-category` (`js/portfolio.js`)
  - Responsive: Horizontal layout at 768px+, stack on 480px
- **Forms:** Full-width on mobile, 2-column layout at 768px+, real-time validation, error states (`js/contact.js`)
  - Inputs: Full-width on mobile, proper padding
  - Phone input with country code dropdown
  - Submit button: Full-width on mobile (48px+ height)
  - Form status messages with clear success/error states
- **FAQ accordion:** Expand/collapse with `.accordion-item` and `.accordion-header` (`js/contact.js`)
  - Accessible keyboard navigation
  - Smooth animation transitions

## Developer Workflow
- **No build step:** Open `index.html` directly or use VS Code Live Server
- **Testing:** Manually check all breakpoints and browsers (Chrome, Firefox, Safari, Edge)
- **Image management:** Place/optimize images in `assets/images/` (see `assets/images/README.md`). Use `image-guide.html` for placeholder generation during development
- **Portfolio updates:** Add new cards in `portfolio.html` (see card markup example in README)
- **Contact form:** Uses Google Apps Script backend (see `js/contact.js` for endpoint URL)
- **Responsive Testing Checklist:**
  - 📱 **Mobile (480px)**: Form full-width, hamburger menu works, touch targets 44px+, single column portfolio
  - 📱 **Tablet (768px)**: Form two-column, menu still accessible, 2-column portfolio grid, proper spacing
  - 💻 **Desktop (1024px)**: Full layout, max-width container, 3-column portfolio, desktop features active
  - 💻 **Wide (1440px+)**: Optimal spacing, proper alignment, no excessive white space
  - ✅ Test on actual devices: iPhone (375px), iPad (768px), MacBook (1440px)
  - ✅ Test browsers: Chrome, Firefox, Safari, Edge
  - ✅ Test orientation: Portrait & landscape on phones and tablets
  - ✅ Test form submission: Works on all breakpoints
  - ✅ Test menu: Hamburger visible/works on mobile, closes properly
  - ✅ Test portfolio filter: Responsive button layout, grid adjusts
  - ✅ Verify images: Load with lazy loading, scale proportionally
  - ✅ Check accessibility: Keyboard nav, ARIA labels, color contrast

## Naming & Conventions
- **HTML:** Lowercase, hyphens (`project-detail.html`)
- **CSS:** Lowercase, component-based (`button.css`, `hero.css`)
- **JS:** camelCase (`portfolioFilter.js`)
- **Images:** Lowercase, hyphens, descriptive (`hero-laptop-glow.png`)

## Integration & Extensibility
- **No external JS dependencies**; all logic is custom and modular
- **No CMS/admin**; all content is static and updated manually
- **Image formats:** Prefer WebP/GIF for animations, JPG/PNG for photos (optimize for web)

## Known Limitations
- No backend for contact form (uses Google Apps Script serverless backend)
- Static content only (no dynamic/CMS features)
- Portfolio/project updates require manual HTML edits
- Responsive design uses CSS media queries (no JavaScript breakpoint detection)

---

## 🎯 Responsive Design Best Practices

### Standard Breakpoints (Consolidated ✅)
```css
/* Mobile First Approach */
Base styles:              (320px and up)
@media (max-width: 480px)  { } /* Mobile phones */
@media (max-width: 768px)  { } /* Tablets & below */
@media (max-width: 1024px) { } /* Small desktop & below */
@media (min-width: 1025px) { } /* Large desktop & wide screens */
```

### CSS Responsive Patterns
```css
/* Flexible Typography (clamp) */
h1 { font-size: clamp(2.5rem, 5vw, 5rem); } /* Scales smoothly */

/* Flexible Grid Layout */
.grid { grid-template-columns: repeat(auto-fit, minmax(300px, 1fr)); }

/* Container Max-Width */
.container { max-width: 1200px; margin: 0 auto; width: 100%; }

/* Responsive Images */
img { width: 100%; height: auto; loading: lazy; }

/* Touch-Friendly Buttons */
button { min-width: 44px; min-height: 44px; }

/* Mobile-First Media Queries */
@media (max-width: 768px) { /* Add tablet features */ }
@media (max-width: 480px) { /* Add mobile features */ }
```

### Common Responsive Patterns Used
- **Grid**: 3-col (desktop) → 2-col (tablet) → 1-col (mobile)
- **Form**: 2-col grid (desktop) → 1-col (mobile)
- **Menu**: Desktop nav (1024px+) → Hamburger overlay (mobile)
- **Hero**: 100vh (desktop) → 70vh (tablet) → 60vh (mobile)
- **Spacing**: Generous (desktop) → Compact (mobile) using CSS variables

### When Making Changes
1. ✅ Test at **all breakpoints** (480px, 768px, 1024px, 1440px)
2. ✅ Ensure **touch targets minimum 44px × 44px** on mobile
3. ✅ Use **clamp()** for flexible typography
4. ✅ Maintain **mobile-first approach** (base styles for mobile, add for desktop)
5. ✅ Test on **actual devices** if possible (DevTools isn't enough)
6. ✅ Verify **images scale** without distortion
7. ✅ Check **no horizontal scroll** at any breakpoint
8. ✅ Ensure **form inputs accessible** on mobile
9. ✅ Test **menu/navigation** at all sizes
10. ✅ Verify **portfolio grid** responsiveness

### Common Pitfalls to Avoid
- ❌ Using desktop breakpoints (900px, 1200px) instead of standard (480/768/1024)
- ❌ Using `px` for font sizing (use `rem` or `clamp()`)
- ❌ Making touch targets too small (<44px on mobile)
- ❌ Adding desktop-first CSS instead of mobile-first
- ❌ Not testing on actual devices
- ❌ Using overflow: hidden without care on mobile
- ❌ Forgetting `loading="lazy"` on images
- ❌ Not testing form interactions on mobile keyboards

---

## 🛑 Copilot Agent Process Rules (MANDATORY)

1. When the user gives a to-do/command, your only objective is to work specifically on that request and touch no other code.
2. After every request is complete, your description of what you did must mention every single line number that you changed and the file name.
3. Always ensure when the user gives a request there isn't any existing hard code that blocks the request; prioritize scalable and malleable code.

**You must follow these rules for every request, in addition to all other project instructions above.**

---

**For AI agents:**
- Follow the modular JS pattern (one concern per file)
- Use CSS custom properties for all colors/spacing
- Maintain dark theme and blue/coral accent consistency
- Reference `assets/images/README.md` for image requirements
- Test all UI at mobile (480px), tablet (768px), and desktop (1024px+) breakpoints
- Use standard breakpoints consistently across all CSS files
- Ensure minimum 44px × 44px touch targets on all interactive elements
- Use `clamp()` for responsive typography
- Verify images have `loading="lazy"` attribute
- Test form inputs and menu on actual mobile devices when possible

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Bernie-Dramos)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Bernie-Dramos)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
