# Lukas Model Portfolio — Copilot Instructions

## Project Overview
This is a **professional model portfolio website** built with vanilla HTML, CSS, and JavaScript. The site showcases Lukas's work across multiple categories (headshots, campaigns, editorials, videos) with a focus on clean design, fast loading, and intuitive navigation.

**Key characteristics:**
- Single-page scrollable site with fixed navigation
- Responsive photo grid layouts with lightbox viewer
- Video showcase sections with alternating layouts
- Contact section with social media links
- No build tools, frameworks, or package managers—purely static assets

---

## Architecture & File Structure

### Core Files
- **`index.html`** — Main document structure (210 lines)
  - Navigation bar with section links
  - Hero section with profile image
  - Seven content sections: Profile/Measurements, Headshots, London Shoots, Nike Campaign, Shein Work, Other Creative Work, Videos, Contact
  - Lightbox container and video item markup

- **`script.js`** — Event handlers (70 lines)
  - Lightbox system: click image → modal overlay with enlarged photo
  - Navbar scroll effect: transitions styling at 50px scroll threshold
  - Smooth scroll navigation: intercepts anchor links with `scrollIntoView()`
  
- **`style.css`** — Layout & styling (257 lines)
  - CSS Grid for responsive photo galleries (`grid-template-columns: repeat(auto-fit, minmax(280px, 1fr))`)
  - Fixed sticky navbar with z-index layering
  - Photo grid hover effects (`transform: scale(1.02)`)
  - Lightbox overlay (`position: fixed`, z-index 2000)
  - Video section alternating layout with `.reverse` class direction trick

- **`images/`** — Asset directory
  - Mixed formats: `.jpg`, `.JPG`, `.PNG`, `.HEIC`, `.mp4`, `.MP4`
  - Note: File naming inconsistencies (uppercase/lowercase extensions, spaces in filenames like "first shoot1.MP4")

---

## Design Patterns & Conventions

### Layout & Grid System
- **Section spacing:** Consistent `padding: 6rem 6%` with horizontal padding scaling
- **Grid photos:** `repeat(auto-fit, minmax(280px, 1fr))` for responsive masonry
- **Photo height:** Fixed `height: 420px` with `object-fit: cover` to maintain aspect ratio
- **Alternating backgrounds:** `.bg-light` applies `background: #f9f9f9` to odd sections

### Styling Patterns
- **Typography:** Montserrat font stack with weight variants (400, 500, 600, 800)
- **Color palette:** Minimalist (#1a1a1a text, #f9f9f9 accents, #ffffff background)
- **Transitions:** Consistent `0.3s ease` or `0.4s ease` for hover/scroll effects
- **Letter spacing:** Ultra-wide for headlines (3-6px) to match luxury brand aesthetic

### JavaScript Conventions
- **DOMContentLoaded wrapper:** All JS logic runs after page fully loads
- **Organized sections:** Code divided into numbered comment blocks (1. LIGHTBOX, 2. NAVBAR, 3. SMOOTH SCROLL)
- **Class toggles:** `.active` class pattern for showing/hiding UI elements
- **Body overflow control:** Lightbox prevents background scrolling with `document.body.style.overflow`

---

## Common Development Tasks

### Adding New Content Sections
1. Create `<section class="section" id="section-id">` in `index.html`
2. Add section title `<h2>`, description `<p class="section-description">`
3. Include `.photo-grid` for images or `.video-item` for videos
4. Add nav link in navbar: `<li><a href="#section-id">Label</a></li>`
5. Lightbox and smooth scroll auto-activate—no JS changes needed

### Adding Images
- Supported formats: JPEG, PNG, HEIC (avoid spaces in filenames for consistency)
- Photo grid automatically handles aspect ratio with `object-fit: cover`
- **Rotated images fix:** Apply `.rotate-fix` class (see `london1.heic` example)

### Adjusting Spacing & Sizes
- Hero section height: `.hero { height: 100vh; }`
- Section padding: `.section { padding: 6rem 6%; }` (adjust rem values globally)
- Photo grid gap: `.photo-grid { gap: 1.2rem; }` (reduce for tighter layouts)
- Navbar top padding: `nav { padding: 1.2rem 6%; }` (scales with responsiveness)

### Navbar Scroll Behavior
- Scroll threshold: 50px (`window.scrollY > 50`)
- Active state: Adds drop shadow and solid white background
- All nav links are anchor-based (`href="#id"`)—smooth scroll auto-engages

---

## Important Quirks & Edge Cases

### Image File Naming
- Mixed case extensions present (`.JPG`, `.jpg`, `.HEIC`, `.heic`)
- Filenames with spaces: `"first shoot1.MP4"`, `"headshot video.mp4"` work but not ideal
- Recommendation: Standardize to lowercase extensions and hyphens over spaces

### Rotated HEIC Images
- London shoot images use `.HEIC` format and may render rotated
- Solution: Apply `.rotate-fix` class with CSS `transform: rotate(90deg)` + `object-fit: contain`
- This is a known workaround for HEIC browser support inconsistencies

### Video Controls
- Videos use native `<video>` element with `controls` attribute
- Browser default controls apply—no custom player built
- File sizes not optimized; consider compressing for production

### Grid Responsiveness
- `minmax(280px, 1fr)` ensures at least 280px columns; adapts tablet/mobile
- No explicit mobile breakpoints—pure CSS Grid flexibility
- At very small screens, consider reducing min-width or adjusting gap

---

## Performance & Best Practices

- **No dependencies:** Vanilla JS only—browser compatibility excellent across older devices
- **Asset optimization:** Images are not compressed; recommend WebP alternatives for production
- **CSS efficiency:** Minimal selectors; all styles applied inline (no unused CSS)
- **Scroll performance:** Scroll event listener active continuously—acceptable for this use case
- **Lightbox UX:** Prevents background scroll; clicking outside modal closes it

---

## Quick Editing Checklist

✓ Adding a new section? Update navbar links + section markup + CSS spacing if needed  
✓ Updating contact info? Edit `.contact-item` spans in `index.html`  
✓ Changing colors? Search `#1a1a1a`, `#f9f9f9`, `#ffffff` in `style.css`  
✓ Adjusting hero image? Change `.hero-img` path in `index.html`  
✓ Need mobile optimizations? Test grid responsiveness and reduce photo heights on smaller screens  

---
> Source: [trkaydo44/MODEL.LUKAS](https://github.com/trkaydo44/MODEL.LUKAS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-29 -->
