# Copilot Instructions for Gitfolio

## Project Overview
Gitfolio is a **single-page portfolio template** for developers. All user-customizable content lives in `app/page.tsx`. This is a beginner-friendly template—keep changes simple and well-commented.

## Architecture

### Single-File Content Model
- **`app/page.tsx`** - Contains ALL portfolio content and components (nav, hero, projects, about, contact, footer)
- Components are defined at bottom of file: `ProjectCard`, `SocialLink`, `ArrowRightIcon`, `ArrowUpRightIcon`
- No separate component files—intentionally flat for beginner accessibility

### GitHub Pages Deployment Pattern
This project uses a **dual-environment basePath** pattern for local dev + GitHub Pages:

```tsx
// In page.tsx - for images
const basePath = process.env.NODE_ENV === "production" ? "/gfbs3-portfolio-demo" : "";
src={`${basePath}/me.png`}

// In next.config.ts - for routing
basePath: isProd ? "/gfbs3-portfolio-demo" : "",
```

**Critical**: When users fork this repo, they must update the repo name in BOTH files.

## Design System

### Color Palette (Tailwind classes)
| Color | Primary Use | Tailwind Classes |
|-------|-------------|------------------|
| Cyan | Links, accents, tech | `cyan-400`, `cyan-500`, `cyan-700` |
| Fuchsia | Highlights, emphasis | `fuchsia-400`, `fuchsia-500`, `fuchsia-600` |
| Yellow | CTAs, warnings | `yellow-400`, `yellow-500` |
| Purple | Gradients | `purple-400`, `purple-500` |

### Glow Effects Pattern
```tsx
// Text glow
className="drop-shadow-[0_0_8px_rgba(34,211,238,0.6)]"

// Box glow on hover  
className="hover:shadow-[0_0_20px_rgba(34,211,238,0.4)]"
```

### ProjectCard Component
```tsx
<ProjectCard 
  title="PROJECT_NAME"      // UPPERCASE_SNAKE_CASE convention
  description="..."
  tags={["REACT", "API"]}   // Uppercase tags
  color="cyan"              // cyan | fuchsia | purple | yellow
  href="https://..."
/>
```

## Key Files

| File | Purpose |
|------|---------|
| `app/page.tsx` | All content + components |
| `app/layout.tsx` | Metadata, fonts (Geist) |
| `app/globals.css` | Tailwind imports only |
| `next.config.ts` | Static export + basePath |
| `.github/workflows/deploy.yml` | GitHub Pages CI/CD |
| `public/me.png` | Profile image location |

## Commands
```bash
npm run dev    # Local development at localhost:3000
npm run build  # Production build (static export to /out)
npm run lint   # ESLint check
```

## When Helping Users

1. **Content changes** → Edit `app/page.tsx` only
2. **Adding images** → Place in `/public`, use `${basePath}/filename` pattern
3. **Deployment issues** → Check basePath matches repo name in both config files
4. **Styling** → Use existing Tailwind classes; maintain cyberpunk aesthetic
5. **New sections** → Follow existing section pattern with `id` for nav linking

---
> Source: [github-samples/gitfolio](https://github.com/github-samples/gitfolio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-03 -->
