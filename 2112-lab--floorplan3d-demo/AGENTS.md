- Always use http://localhost:3000/ to test in the browser, which should be assumed to be already running. So no "npm run dev" command is necessary.

## Local Development Setup

### Floorplan3D Module Development
- This project uses a **nested repository** setup for the `@2112-lab/floorplan3d` npm module
- The module source code is located in `./floorplan3d/` directory within the project root
- The `floorplan3d/` directory has its own `.git` repository, separate from the main app
- When `LOCAL_DEV=true` is set in `.env`, the app uses the local module instead of the published npm package

### Repository Structure
- `floorplan3d-demo/` - Main Nuxt.js application (has own .git repository)
- `floorplan3d-demo/floorplan3d/` - Nested module repository (has own .git repository)
- Both repositories have independent version control and commit history
- Changes to the module require commits to the module repository
- Changes to the app require commits to the app repository

### Module Resolution
- In development mode: Points to `floorplan3d/src/index.js` (source files for immediate changes)
- In production mode: Points to `floorplan3d/dist/bundle/index.js` (built bundle)
- The alias is configured in `nuxt.config.js` under the webpack extend function

### Development Workflow
- Changes to `floorplan3d/src/` files are picked up immediately by the Nuxt app
- If you need to test the built version, run `npm run build` in the `floorplan3d/` directory
- The module is transpiled by Nuxt's build process, so no separate build step is required for development
- When making changes that span both repositories, commit to each repository separately

### Git Workflow
- **Module changes**: `cd floorplan3d/` then commit and push to module repository
- **App changes**: Stay in root directory, commit and push to app repository
- **Both repositories**: Handle commits separately when changes affect both
- Use `git remote -v` to check which repository you're currently in

### Key Files
- `floorplan3d/src/index.js` - Main module entry point (development)
- `floorplan3d/dist/bundle/index.js` - Built bundle (production)
- `.env` - Contains `LOCAL_DEV=true` to enable local development mode
- `nuxt.config.js` - Contains the webpack alias configuration for module resolution

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/2112-lab)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/2112-lab)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
