# Seelen Navigatoren Website - AI Agent Instructions

## Architecture Overview
This is a static HTML website with dynamic content loaded via JavaScript. No build process or frameworks - pure vanilla JS fetching JSON data and HTML components.

- **Static Pages**: Core content in HTML files (index.html, brio.html, etc.) with shared CSS (main.css).
- **Dynamic Data**: Events and videos loaded from `data/events.json` and `data/videos.json` via fetch in `js/events.js` and `js/videos.js`.
- **Components**: Reusable HTML snippets (e.g., `components/brevo-form.html`) loaded into page containers using fetch in loaders like `js/brevo-form-loader.js`.
- **Styling**: Dark theme (#0b3540 background), responsive with media queries, fonts: Poppins (body), Inter (headings).
- **Deployment**: GitHub Actions FTP upload to DomainFactory hosting (no local build needed).

## Key Patterns
- **Data Loading**: Use relative paths for JSON (e.g., `'data/events.json'`) to work on subpages. Handle fetch errors gracefully.
- **Component Inclusion**: Load HTML components into `<div id="component-container">` via `fetch('components/file.html').then(r => r.text()).then(html => container.innerHTML = html)`.
- **Event Formatting**: Dates in `YYYY-MM-DD` format, formatted to German locale (e.g., "Mi., 21.01.2026") using `Intl.DateTimeFormat`.
- **YouTube Embeds**: Use `.yt-consent` class with `data-yt-src` attribute; consent managed via `js/cookies.js` with localStorage key `'sn_consent_v1'`.
- **Links**: External links use `target="_blank" rel="noopener noreferrer"` for security.
- **Content Language**: Primarily German; maintain consistent terminology (e.g., "Termine" for events, "Videos / Podcasts").

## Data Structures
- **Events JSON**: Array of objects with `id`, `category`, `title`, `date`, `time`, `location` (name/url), `leader`, `leader-link`, `detailUrl`, `notes`.
- **Videos JSON**: Array of objects with `id`, `title`, `date`, `url` (YouTube embed).

## Development Workflow
- Edit HTML/CSS/JS directly; no compilation.
- Test locally by opening HTML files in browser (relative paths work).
- Add new events/videos by updating JSON files with consistent structure.
- For new components, create HTML in `components/` and loader JS in `js/` following existing patterns.
- Deployment: Push to GitHub main branch triggers automatic FTP upload.

## Conventions
- File naming: Lowercase, hyphens (e.g., `ehrliches-mitteilen.html`).
- CSS: Class-based styling, no preprocessors.
- JS: IIFE patterns for encapsulation, async/await for fetches.
- Images: Stored in `Images/` folder, referenced relatively.
- Meta tags: Include Open Graph and Twitter Card tags for social sharing.

## Common Tasks
- **Add Event**: Append to `data/events.json` with unique `id` (e.g., "mg-2026-01-21"), ensure date/time/location format.
- **Add Video**: Append to `data/videos.json` with YouTube embed URL.
- **Update Form**: Edit `components/brevo-form.html` (Brevo newsletter integration).
- **Style Changes**: Modify `main.css` or add page-specific CSS links.

Reference files: `data/events.json`, `data/videos.json`, `components/brevo-form.html`, `js/events.js`, `js/videos.js`, `main.css`.</content>
<parameter name="filePath">/Users/brio/Documents/Seelen-Navigatoren/Website/.github/copilot-instructions.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/briomusic)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/briomusic)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
