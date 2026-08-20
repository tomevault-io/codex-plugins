# Project-Me — Copilot Instructions

## Overview

Interactive SvelteKit 5 portfolio app integrating GitHub, Spotify, and Steam APIs. Designed as a publishable Svelte library (`svelte-package`) and a standalone showcase app.

## Architecture

### Schema-First Types

Types are defined in YAML (`docs/types/*.yaml`) and code-generated into `src/generated/types/` via **type-crafter**.

- **Rule**: ALL shared types and configuration schemas must be defined in YAML.
- **Workflow**: Edit YAML -> `pnpm generate` -> Use generated types.
- **Never** hand-write types in `src/generated/` or manually type configuration files without a backing schema.

### Source-Based Polymorphism

The `Source` union (`"github" | "spotify" | "games"`) from `src/generated/types/Projects.ts` drives most of the app: API routing (`/api/infos?source=...`), navigation menus, theme selection, and UI rendering. When adding a new data source, update the YAML schema, regenerate types, and wire through configuration, services, and stores.

### Client ↔ Server Split

- **`src/lib/services/`** — Server-side services that call external APIs (GitHub, Spotify, Steam, mailer). Each uses `APICaller` with generated decoders.
- **`src/lib/client/`** — Browser-side fetchers that call internal SvelteKit API routes. Check store state (`null` = unfetched) before fetching.
- **`src/lib/server/`** — Server-only utilities: `APIResponseHandler` (builder pattern for standardized responses) and domain mappers.

### Component Architecture

- **Modular Design**: Break down complex UIs into small, single-purpose components (e.g., `Badge`, `Button`, `Card`).
- **Data-Driven**: Components should receive data via props, typically sourced from typed configuration files.
- **Icons & SVGs**: Use dedicated Svelte components for all icons and graphical assets (e.g., `src/lib/components/icons/PackageIcon.svelte`). **NEVER** use inline `<svg>` blocks directly inside feature components or pages. Always extract them to the `$components/icons` directory.
- **Composition**: Build "Section" components (e.g., `Technical.svelte`) by composing smaller UI atoms.

### Writing CSS, Themes, and Class Names

- **Theme Variables Only**: **NEVER** hardcode hex codes (e.g., `#FFFFFF`), `rgba()` strings, or pixel sizes directly in component `<style>` blocks or feature SCSS files. Every color must exist as CSS custom properties (`var(--color-...)`) inside `src/css/theme.scss`.
- **Class naming**: Class names should be in kebab-case and should be descriptive of the element's purpose or content. For example, use `class="profile-card"` instead of `class="card1"`.
- **Global Abstractions**: We can use generic class names whose styles would be defined in `src/css/style.scss`. For example, a generic `class="button"` defined in `style.scss` can be used across multiple components.
- **Typography & Scale**: Utilize `clamp()` for responsive, fluid typography (especially for massive Hero headers) to ensure seamless scaling between mobile and desktop without excessive media queries.

### API Route Pattern

All routes in `src/routes/api/` follow this template:

```typescript
export async function GET({ url, request }: RequestEvent) {
  const tag = "GET /api/...";
  logger.logServerRequest(tag, { ... });
  try {
    // business logic using services + decoders
    response = APIResponseHandler.successResponse("message", data);
  } catch (err) {
    logger.logException(tag, String(err));
    response = APIResponseHandler.badRequestResponse(String(err));
  }
  logger.logServerResponse(tag, { ... });
  return APIResponseHandler.toResponse(response);
}
```

Use `APIResponseHandler` static factories (`successResponse`, `badRequestResponse`, `notFoundResponse`, etc.) — never construct raw `Response` objects in API routes.

### Stores

Svelte `writable` stores with typed state objects and dedicated updater functions. Convention: `null` means "not yet fetched." See `src/lib/stores/site.ts` (theme/source/loader) and `src/lib/stores/skills.ts` (cached API data).

## Path Aliases

Defined in both `svelte.config.js` and `vite.config.ts`. Always use these in imports:

| Alias            | Path                    |
| ---------------- | ----------------------- |
| `$generated`     | `src/generated`         |
| `$services`      | `src/lib/services`      |
| `$stores`        | `src/lib/stores`        |
| `$configuration` | `src/lib/configuration` |
| `$components`    | `src/lib/components`    |
| `$pages`         | `src/lib/pages`         |
| `$constants`     | `src/lib/constants`     |
| `$server`        | `src/lib/server`        |
| `$client`        | `src/lib/client`        |
| `$models`        | `src/lib/models`        |
| `$css`           | `src/css`               |

## Key Libraries

- **vergins** — UI component library (Navbar, Flyer, TimeLine, etc.) and its shipped CSS
- **lilac-typescript** — Structured logger (`ProcessLogger`); use `logger.logServerRequest`, `logServerResponse`, `logException`, `logExternalApiRequest/Response`
- **type-decoder** — Runtime decoding primitives (`decodeString`, `decodeNumber`, `isJSON`, etc.) used inside generated decoders
- **type-crafter** — CLI that generates TypeScript types + decoders from YAML schemas
- **chart.js** — Doughnut charts in `src/lib/components/Charts/`
- **nodemailer** — SMTP email (Gmail) in the mailer service

## Configuration

- **Server secrets** (`src/lib/configuration/config/index.ts`): API keys via `process.env` + `dotenv`
- **Client config** (`src/lib/configuration/config/client/index.ts`): Public values via `import.meta.env.VITE_*`
- **Page content** (`src/lib/configuration/{home,about,skills,navigation,github}.ts`): Static data arrays for UI rendering.
  - **Rule**: These files must be typed using generated types (e.g., `TechnicalCategory[]`, `HomeConfig`).
  - **Pattern**: `export const myConfig: GeneratedType = [...]`

## Commands

| Command         | Purpose                            |
| --------------- | ---------------------------------- |
| `pnpm dev`      | Dev server on port 4366            |
| `pnpm build`    | Production build + package         |
| `pnpm generate` | Regenerate types from YAML schemas |
| `pnpm check`    | TypeScript + Svelte type checking  |

## Conventions

- Components use **Svelte 5** runes syntax
- SCSS for styling (`src/css/style.scss`, `src/css/theme.scss`) using **CSS custom properties** (`var(--color-primary)`, `var(--space-4)`). Avoid hardcoded values completely.
- **Strictly** follow the design system tokens found in `theme.scss`.
- Structured logging wraps every service call and API route — always include a `tag` string
- `APICaller` uses a **builder pattern** with a generic decoder callback — follow existing services as templates
- The `hooks.server.ts` handles CORS preflight and 404 redirects (pages → `/not-found`, API → structured error response)

## Design & Aesthetics

- **Modern & Premium**: The app targets a premium, dark-mode-first aesthetic with dynamic hover states, subtle glowing shadows, and aggressive, highly-legible typography.
- **UI Architecture**: Utilize layered depth using `#000000`, `#0a0a0a`, `#111111`, and `#1a1a1a` backgrounds with subtle dashed or `0.05` opacity white borders for segmentation.
- **Consistency**: Refer to `theme.scss` for established coloring schemes and font configurations. The app must be fully responsive and look good on both desktop and mobile devices.

- Refer (docs/newdesign.png) for the new design mockup. The app should be a single-page portfolio with sections for GitHub repos, Spotify top tracks, and Steam games, plus an about page and contact form. Use the existing `vergins` (wherever possible and can be ignored in case feature is kind of not possible) components where possible and create new ones as needed to match the design.
- We will follow somewhat like variable coloring schemes and font also those should be consistent with the design. The app should be responsive and look good on both desktop and mobile devices.

---
> Source: [AmreetKumarkhuntia/project-me](https://github.com/AmreetKumarkhuntia/project-me) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-20 -->
