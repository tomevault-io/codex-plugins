# Story Game AI

Story Game AI is a Next.js 15.5.2 application that creates interactive AI-powered story games. It uses Google AI for story and image generation, React 19 for the UI, Tailwind CSS 4 for styling, and TypeScript for type safety.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Quick Start
1. `pnpm install` (20 seconds)
2. `pnpm dev` (starts in 1 second)
3. Open http://localhost:3000
4. Verify the Story Game AI interface loads with sidebar settings and "Start Game" button

## Working Effectively

### Bootstrap, build, and test the repository:
- `pnpm install` -- takes 20 seconds. Installs 429 packages.
- `pnpm lint` -- takes <1 second. ALWAYS run before committing.
- `pnpm build` -- takes 30 seconds. NEVER CANCEL. Set timeout to 90+ minutes.
  - **CRITICAL**: Build WILL FAIL due to Google Fonts network restrictions with error "Failed to fetch `Geist` from Google Fonts". This is expected in restricted environments.
  - **WORKAROUND**: Temporarily comment out font imports in `src/app/layout.tsx` if production build is required.
- `pnpm dev` -- takes 1 second to start, 15 seconds for first compile. NEVER CANCEL. Set timeout to 30+ minutes.
- `pnpm start` -- starts production server in <1 second.

### Run the application:
- **Development**: `pnpm dev` then open http://localhost:3000
- **Production**: `pnpm build && pnpm start` then open http://localhost:3000
- **CRITICAL**: Application requires GOOGLE_GENERATIVE_AI_API_KEY environment variable to generate stories. Without it, the game will show error messages but the UI will still function.

### Environment Setup:
- **Required**: Node.js 18+ (tested with 20.19.4)
- **Required**: pnpm (install with `npm install -g pnpm`)
- **Optional**: Google AI API key from https://ai.dev for full functionality

## Validation

### Always manually validate any new code changes:
1. **ALWAYS run `pnpm lint`** -- Must pass with no errors before committing
2. **Start dev server** -- `pnpm dev` and verify http://localhost:3000 loads
3. **Test UI functionality**:
   - Verify sidebar story settings are editable
   - Confirm "Start Game" button works (will show error without API key, this is expected)
   - Check responsive design on different screen sizes
4. **Build validation** -- Run `pnpm build` to test production build capability
   - **EXPECTED**: Build will fail with "Failed to fetch `Geist` from Google Fonts" - this is normal in restricted environments
   - Document this as "Build fails due to expected network restrictions" in your changes

### NEVER CANCEL BUILD OR TEST COMMANDS:
- Builds may take 30+ minutes in some environments
- Always set timeouts to 90+ minutes for build commands
- Always set timeouts to 30+ minutes for development server startup

### Functional Testing Scenarios:
After making changes, always test these scenarios:
1. **Application Startup**: Navigate to http://localhost:3000, verify welcome screen appears with dark theme
2. **Story Configuration**: Modify settings in sidebar (genre, style, narrator role, initial situation, invite phrases, image style)
3. **Game Start Flow**: Click "Start Game" and verify conversation interface appears
4. **Error Handling**: Verify graceful degradation when API key is missing (shows error but UI remains functional)
5. **Responsive Design**: Test on different viewport sizes using browser dev tools
6. **Take Screenshot**: Document any UI changes for stakeholders

## Common Tasks

### Repository Structure:
```
src/
  app/
    api/
      generate-image/     # Image generation API endpoint
      generate-story/     # Story generation API endpoint
    components/           # Game-specific components
    hooks/               # Custom React hooks
    layout.tsx           # Main layout with fonts
    page.tsx            # Home page component
  components/            # Shared UI components
    ui/                 # UI primitives (buttons, inputs, etc.)
  lib/                  # Utility functions, constants, prompts
  types/                # TypeScript type definitions
```

### Key Components:
- **src/app/components/**: Game-specific components (input, loader, message, sidebar, story config)
- **src/components/**: Shared components (conversation, image, reasoning, etc.)
- **src/components/ui/**: UI primitives (avatar, badge, button, input, etc.)
- **src/lib/**: Utility functions, constants, and prompts
- **src/types/**: Type definitions for game and settings

### API Routes:
- **/api/generate-story/**: Handles story generation requests using Google AI
- **/api/generate-image/**: Handles image generation requests using Google AI

### Important Files:
- **biome.json**: Linting and formatting configuration
- **package.json**: Dependencies and scripts (use pnpm, not npm)
- **next.config.ts**: Next.js configuration
- **tailwind.config.js**: Tailwind CSS configuration
- **tsconfig.json**: TypeScript configuration

### Font Issues and Workarounds:
The application uses Google Fonts (Geist and Geist Mono) which may fail to load in restricted network environments:
- **Development**: App falls back to system fonts automatically
- **Production Build**: May fail completely. If needed, temporarily comment out font imports in `src/app/layout.tsx`
- **Always restore original fonts** after testing builds in restricted environments

### Common Commands Reference:
```bash
# Install dependencies (required first step)
pnpm install

# Development server (most common)
pnpm dev

# Linting (run before every commit)
pnpm lint

# Format code
pnpm format

# Production build
pnpm build

# Start production server
pnpm start
```

### Frequent Gotchas:
1. **Use pnpm, not npm** -- The project is configured for pnpm workspace
2. **Google Fonts may fail** -- This is expected in restricted environments
3. **API key required for functionality** -- App works without it but shows errors
4. **Biome, not ESLint** -- Uses Biome for linting and formatting
5. **Turbopack enabled** -- Uses Next.js Turbopack for faster builds

### Package Dependencies:
- **@ai-sdk/google**: Google AI integration for story/image generation
- **next**: React framework (version 15.5.2)
- **react**: UI library (version 19.1.0)
- **tailwindcss**: CSS framework (version 4)
- **@biomejs/biome**: Linting and formatting
- **typescript**: Type checking

### Environment Variables:
Create `.env.local` with:
```
GOOGLE_GENERATIVE_AI_API_KEY=your_google_ai_api_key_here
```

## Troubleshooting

### Build Failures:
1. **Google Fonts error**: EXPECTED in restricted networks. Error: "Failed to fetch `Geist` from Google Fonts"
   - **Solution**: Temporarily comment out `import { Geist, Geist_Mono } from "next/font/google";` in `src/app/layout.tsx`
   - **Restore fonts** after build testing is complete
2. **Shiki dependency error**: Run `pnpm add shiki` (already resolved in this repository)
3. **Network timeouts**: Increase timeout values, do not cancel builds

### Runtime Issues:
1. **Story generation fails**: Check API key configuration
2. **Images not loading**: Verify Google AI API has image generation permissions
3. **UI not responsive**: Check Tailwind CSS classes and viewport meta tag

### Performance:
- **First load**: ~15 seconds for full compilation
- **Hot reload**: <1 second for most changes
- **Build time**: ~30 seconds in optimal conditions
- **Package install**: ~20 seconds

Always validate your changes work correctly by running through the complete user workflow before committing.

---
> Source: [mtmarctoni/story-game-ai](https://github.com/mtmarctoni/story-game-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-04 -->
