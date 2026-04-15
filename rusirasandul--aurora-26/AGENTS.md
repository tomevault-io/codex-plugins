# Aurora 2026 Website - AI Agent Instructions

## Project Overview
Single-page React website for Aurora 2026, an inter-university tech conference on Social Engineering and Agentic AI. The site features a dark theme with glowing orange/amber accents creating an "Aurora" visual effect.

**Stack**: React 18 + TypeScript + Vite + Tailwind CSS + Lucide Icons

## Architecture & Structure

### Single-Component Design
The entire UI lives in [src/App.tsx](../src/App.tsx) as a monolithic `AuroraConference` component. No separate component files, routing libraries, or state management. This is intentional for simplicity.

**Key sections** (in order):
- Fixed glassmorphic navigation with mobile menu toggle
- Hero section with gradient text and stats
- Infinite-scrolling gallery (CSS animations)
- Leadership board (Executive + 6 Committees)
- Footer with social links

### State Management
Single `useState` hook for mobile menu toggle (`isMenuOpen`). Committee data is stored as a const array within the component.

## Design System & Conventions

### Color Palette (Tailwind)
- **Base**: `bg-neutral-950` (deep void black)
- **Primary Accent**: `orange-500`, `orange-400`, `amber-300` gradients
- **Glass Effects**: `bg-black/80 backdrop-blur-md` for nav, `bg-white/5` for cards
- **Borders**: `border-white/10` or `border-white/5` for subtle separation
- **Glow Effects**: `shadow-[0_0_20px_rgba(249,115,22,0.4)]` for buttons/elements

### Typography
- Font: Inter (loaded via Google Fonts in `index.html`)
- Tracking: Wide tracking on small labels (`tracking-widest`)
- Gradients: Use `text-transparent bg-clip-text bg-gradient-to-r from-orange-500 via-amber-200 to-orange-500` for hero text

### Spacing & Layout
- Max-width container: `max-w-7xl mx-auto px-4 sm:px-6 lg:px-8`
- Section padding: `py-24` for major sections, `py-12` for gallery
- Grids: Use `grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3` patterns

## Key Patterns & Techniques

### Infinite Scroll Gallery
Uses CSS keyframe animation with duplicated content:
```css
@keyframes scroll {
  0% { transform: translateX(0); }
  100% { transform: translateX(-50%); }
}
```
Content is doubled (mapped with `[1,2]`) to create seamless looping effect. Animation is in `<style>` tag within component.

### Hover Effects
- Cards: `border-white/5 hover:border-orange-500/50 transition-all`
- Buttons: Scale on hover (`hover:scale-105`) with glow shadows
- Images: Grayscale by default, full color on hover (`grayscale hover:grayscale-0`)

### Responsive Strategy
- Mobile-first with `md:` and `lg:` breakpoints
- Mobile menu: Hamburger icon toggles menu (not implemented in current code)
- Flex direction switches: `flex-col sm:flex-row`

## Development Workflow

### Commands
- **Dev server**: `npm run dev` (Vite serves on port 5173)
- **Build**: `npm run build` (TypeScript check + Vite build)
- **Preview**: `npm run preview` (Preview production build)

### No Hot Module Replacement Issues
Vite HMR works out of the box. Changes to `App.tsx` or styles reflect instantly.

## Content Structure

### Committee Data Format
```typescript
const committees = [
  {
    name: "Committee Name",
    heads: ["Person 1", "Person 2"],
    members: ["Member 1", "Member 2", ...] // Always 6 members
  }
]
```

### Stats Section
Four key metrics displayed in hero: `{ label, value }` objects for Speakers, Universities, Attendees, Date.

## Styling Notes

### Custom CSS in Component
Inline `<style>` tag contains:
- Scroll animation keyframes
- Scrollbar hiding utility (`.scrollbar-hide`)

### Tailwind Extensions
- Font family extended with Inter in [tailwind.config.js](../tailwind.config.js)
- Custom animations defined in [src/index.css](../src/index.css) (`animate-fade-in`)

## Common Tasks

**Adding a new section**: Insert between existing sections in `App.tsx`, follow the pattern of section wrapper with `id` for anchor links, use `max-w-7xl mx-auto` container.

**Updating committee members**: Edit the `committees` array in [src/App.tsx](../src/App.tsx#L11-L18).

**Changing theme colors**: Update Tailwind classes (search/replace `orange-500` with new color), modify glow shadows in `shadow-[...]` utilities.

**Adding icons**: Import from `lucide-react` at top of `App.tsx`, use inline with size prop (e.g., `<Instagram size={18}/>`).

## Dependencies
- **Production**: React 18.3, React-DOM 18.3, Lucide React 0.344
- **Dev**: Vite 5.4, TypeScript 5.6, Tailwind CSS 3.4, @vitejs/plugin-react 4.3

No other libraries. No routing, no API calls, no backend integration.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rusirasandul) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
