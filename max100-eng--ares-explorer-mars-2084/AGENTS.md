# Ares Explorer - AI Agent Instructions

## Project Overview
**Ares Explorer** is a Mars travel planning web application set in 2084. Users plan futuristic Mars trips with AI-powered itineraries, spacecraft selection, colony visualization, and interactive mapping. The application is built with React + TypeScript and uses Vite as the build tool, with Tailwind CSS for styling.

### Key Characteristics
- **Language**: Spanish UI with immersive Mars/space theme
- **Target**: PWA (Progressive Web App) with offline support via service workers
- **Styling**: Tailwind CSS with custom Mars theme (reds/oranges + space blacks)
- **Fonts**: Orbitron (display), Rajdhani (body text) - futuristic aesthetic
- **Deployment**: Vercel with manifest configuration

## Architecture

### Component Structure (`/components`)
Components are functional, likely using hooks. Examples from the planned structure:
- **UI Components**: `Button.tsx` - reusable styled button with Mars theme
- **Feature Components**: 
  - `SpacecraftSelection.tsx` - spacecraft picker for trip planning
  - `ItineraryDisplay.tsx` - AI-generated trip itinerary display
  - `MarsMap.tsx` - interactive Mars map visualization
- **Layout Components**: `StarBackground.tsx` (background effect), `VideoVisualizer.tsx` (media display)
- **Authentication**: `AuthModal.tsx` - user auth flow
- **Branding**: `AppIcon.tsx` - app logo/icon component

### Service Layer (`/services`)
Encapsulates external integrations:
- **`authService.ts`** - Authentication (likely Firebase/Auth0 or custom)
- **`geminiService.ts`** - Google Gemini AI integration for itinerary generation

### Root Configuration Files
- **`types.ts`** - Shared TypeScript interfaces/types (auth state, itinerary, spacecraft, etc.)
- **`constants.ts`** - Magic strings, API keys, theme values, default data
- **`App.tsx`** - Main component routing/layout
- **`service-worker.js`** - PWA offline caching strategy

## Key Patterns & Conventions

### Styling Approach
- Use Tailwind CSS classes directly in JSX
- Leverage custom theme extension in `tailwind.config`:
  - Mars colors: `text-Mars-500`, `bg-Mars-900`
  - Space colors: `bg-espace-800`, `text-espace-900`
  - Custom fonts: `font-display` (Orbitron), `font-sans` (Rajdhani)
- Example: `<button className="bg-Mars-500 font-display text-espace-900">Book Trip</button>`

### Data Flow
1. **User Action** → Component state (React hooks)
2. **Service Call** → `geminiService.ts` generates itinerary OR `authService.ts` handles auth
3. **State Update** → Component renders updated itinerary/map
4. **Cache** → Service worker caches trip data for offline access

### API Integration Pattern
- Services are imported and called within components
- Expected pattern: `const itinerary = await geminiService.generateItinerary(tripParams)`
- All async operations should handle loading/error states in component UI

### PWA & Manifest Configuration
- `manifest.json` configures PWA metadata (app name, icons, start URL)
- Service worker pre-caches critical assets
- Offline fallback page: cached version of last viewed itinerary

## Development Workflow

### Build & Run
```bash
npm run dev      # Start Vite dev server (HMR enabled)
npm run build    # Production build with tree-shaking
npm run preview  # Preview built output locally
```

### Dependencies (Expected)
- **React 18+** with hooks
- **TypeScript** for type safety
- **Tailwind CSS 3+** for styling
- **Vite 4+** for fast builds
- **Google Gemini SDK** for AI
- **Auth library** (Firebase or custom)

## Critical Guidance for Agent Implementation

### When Adding New Features
1. **Place components** in `/components` - one file per component
2. **Extract shared logic** to `/services` if calling external APIs
3. **Update `/types.ts`** with new interfaces (e.g., `Trip`, `Itinerary`)
4. **Use constants** from `constants.ts` for API endpoints, AI model names, theme values
5. **Apply theme colors** - always use custom Mars/space color classes, avoid hardcoded colors

### State Management
- Use React hooks (`useState`, `useContext`) for local component state
- For global state (user auth, trip history): consider Context API or store in localStorage for PWA offline support
- Preserve state in service worker cache for offline trips

### Mars Immersion Theme
- All UI should feel futuristic and space-themed
- Use "Mars" and "espace" (space) color palette exclusively
- Typography: headers in Orbitron (display font), body in Rajdhani
- Loading states: star animations, spaceship transitions, colony grid effects

### Spanish Localization
- UI strings in Spanish (e.g., "Planifica tu viaje")
- Date formatting for Spanish locale
- Consider i18n setup if expanding to multiple languages

## File Paths Reference
- Components: `/components/*.tsx`
- Services: `/services/*.ts`
- Types: `/types.ts`
- Constants: `/constants.ts`
- HTML entry: `/index.html` (note: "índex.html" in codebase, accent included)
- Config: `/vite.config.ts`, `/tsconfig.json`, `/vercel.json`

## Red Flags / Common Mistakes to Avoid
- ❌ Hardcoding colors instead of using Tailwind classes (`bg-Mars-500`)
- ❌ Putting API calls directly in components instead of services
- ❌ Missing error handling on Gemini API calls (rate limits, auth failures)
- ❌ Not caching itineraries in service worker (defeats PWA purpose)
- ❌ Using English UI strings (should be Spanish)
- ❌ Importing constants inline instead of from `constants.ts`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/max100-eng) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
