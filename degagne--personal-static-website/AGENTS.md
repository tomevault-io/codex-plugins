# Copilot Instructions for Portfolio Website

## Project Overview
This is a single-page portfolio website for Deric Degagne, a Lead Software Engineer specializing in ETL processing. The site is built with pure HTML and Tailwind CSS v4.1.8, using custom theme configuration and the AOS (Animate On Scroll) library for animations.

## Architecture & Structure
- **Static HTML site**: No build process or bundler - all files served directly
- **Single main page**: [index.html](index.html) (460 lines) contains entire site structure
- **Custom error page**: [500.html](500.html) for server error handling
- **Static assets**: All resources in `public/` directory (images, styles)

## Styling System
The project uses **Tailwind CSS v4** with a custom theme defined in [public/styles/tailwind.css](public/styles/tailwind.css):

```css
@theme {
  --font-display: "Oswald", sans-serif;  /* Headers, titles */
  --font-text: "Montserrat", sans-serif;  /* Body text */
}
```

- **Compiled styles**: [public/styles/style.css](public/styles/style.css) (936 lines) is the generated output
- **Color palette**: Cyan-500/600/700 for accents, slate-200/400/500/900/950 for text/backgrounds
- **Custom fonts**: Oswald (display) and Montserrat (text) loaded from Google Fonts
- **Class usage**: Apply Tailwind utility classes directly in HTML - no custom CSS classes

## Component Patterns

### Section Structure
Each major section follows this pattern:
```html
<div class="w-screen h-full bg-{section} bg-center py-10 justify-center bg-blend-multiply bg-slate-900 bg-cover">
  <div class="h-full w-full relative z-10 text-slate-900 justify-center">
    <div class="w-9/12 h-full mx-auto p-5 sm:p-8 lg:p-16" id="{section-id}" data-aos="fade-up">
      <!-- Content -->
    </div>
  </div>
</div>
```

### Animation Pattern
All animated elements use AOS library initialized at page bottom:
- Add `data-aos="fade-up"` attribute for fade-in animations
- Library initialized via CDN: `<script src="https://unpkg.com/aos@next/dist/aos.js"></script>`

### Skills Display
Two visualization patterns used in the About section:
1. **Circular gauge**: SVG-based circular progress indicators for high-level skills (Hadoop, AWS, IaC)
2. **Progress bars**: Linear bars for technical skills with inline `style="width:X%"` attributes

### Timeline Layout
Work experience uses a vertical timeline with alternating left/right layout:
- Grid-based on desktop (`md:grid grid-cols-9`)
- Stacked on mobile with visual timeline connector
- Current position marked distinctly at top

## Key Conventions
- **Responsive breakpoints**: sm/md/lg/xl follow Tailwind defaults
- **Color naming**: `text-cyan-500` for accent, `text-slate-{weight}` for grays
- **Spacing**: Use Tailwind's standard spacing scale (p-4, p-6, p-10, etc.)
- **Font classes**: `font-display` for headings, `font-text` for body content
- **External resources**: All hosted on CDN (no local dependencies except compiled CSS)

## Making Changes
- **Edit HTML directly**: No preprocessing step required for HTML changes
- **Styling changes**: If modifying Tailwind config, regenerate [public/styles/style.css](public/styles/style.css) from [public/styles/tailwind.css](public/styles/tailwind.css)
- **New sections**: Follow existing section pattern with proper data-aos attributes
- **Icons**: All SVG icons are inline (LinkedIn, GitHub, Email) - no icon library used

## Vue.js Migration (NEW)
This project has been migrated to Vue.js 3 + Vite with AWS Amplify deployment support:

### New Structure
- **Entry point**: `index-vue.html` → `src/main.js` → `src/App.vue`
- **Components**: All sections split into Vue SFCs in `src/components/`
- **Build system**: Vite for fast dev server and optimized production builds
- **State**: Reactive data using Vue's Composition API (`ref`)

### Development Commands
```bash
npm install        # Install dependencies
npm run dev        # Start dev server (localhost:5173)
npm run build      # Build for production (output: dist/)
npm run preview    # Preview production build
```

### Component Data
Skills and experience are now reactive data structures:
- Edit skill values in `src/components/SkillsSection.vue`
- Edit job history in `src/components/ExperienceSection.vue`

### Deployment
AWS Amplify configuration in `amplify.yml` handles:
- Automated builds on git push
- CDN distribution
- SSL certificates
- Custom domain setup

See [README.md](README.md) and [MIGRATION-GUIDE.md](MIGRATION-GUIDE.md) for detailed instructions.

## Legacy Static Version
The original static HTML files (`index.html`, `500.html`) are preserved for reference. The project intentionally had no build system and was served directly by any web server.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/degagne)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/degagne)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
