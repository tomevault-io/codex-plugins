## Required Context

- Always read `/AI_REPO_GUIDE.md` first and follow it
- If `AI_REPO_GUIDE.md` conflicts with `README.md`, `AI_REPO_GUIDE.md` is the authoritative source.

## Project Overview

A web-based Pong game optimized for mobile devices with a retro synthwave aesthetic, featuring single-player AI, local multiplayer, and online multiplayer with global leaderboards.

### Tech Stack
- Language: JavaScript (vanilla, ES6+)
- Framework: None (pure HTML5/CSS3/Canvas)
- Build tool: None (static files, no build step)
- Test framework: Not configured yet
- Backend: Node.js + Express + Socket.io (when implemented)
- Database: PostgreSQL via Supabase (when implemented)

## Current Status

⚠️ **This repository is in the planning/documentation phase.** No implementation code exists yet. All specifications are documented in `docs/prompts/`.

## Quick Commands

### Setup
```bash
# Clone repository
git clone https://github.com/mikejmckinney/pong7.git
cd pong7

# No dependencies to install yet (no package.json)
```

### Verification
```bash
# Verify repository structure
./test.sh
```

Expected output: All checks should pass (21 passed, 0 warnings, 0 failed)

### Build
**Not applicable** - No build step required for vanilla JavaScript.
Once implemented, frontend will be static HTML/JS/CSS files.

### Test
```bash
# Run unit tests
npm test

# Run tests with coverage
npm run test:coverage

# Run E2E tests
npm run test:e2e

# Run backend tests
cd server && npm test
```

### Lint
```bash
# Lint code
npm run lint

# Auto-fix lint issues  
npm run lint:fix
```

### CI/CD
The CI pipeline (`.github/workflows/ci-tests.yml`) runs on every push:
- Lint → Unit Tests → Integration Tests → Build → E2E Tests

**Self-Healing Protocol**: If CI fails, fix issues before marking tasks complete.
See `.github/agents/self-healing.agent.md` for details.

## Project Structure

Key directories:
- `docs/prompts/` - Complete game design documentation (10 files)
- `.github/agents/` - Custom agent definitions
- `.github/prompts/` - Onboarding documentation
- `.github/workflows/` - CI verification workflows

Implementation structure (to be created):
- Root: `index.html`, `manifest.json`, `service-worker.js`
- `js/` - Game logic, physics, controls, AI, networking
- `css/` - Styles, animations, responsive design
- `assets/` - Sounds, fonts, images
- `server/` - Node.js backend for multiplayer

## Key Files

| File | Purpose |
|------|---------|
| `AI_REPO_GUIDE.md` | Developer quick reference and repository overview |
| `README.md` | Project introduction and roadmap |
| `AGENTS.md` | AI agent workflow guidelines |
| `docs/prompts/README.md` | Main project documentation with implementation phases |
| `docs/prompts/01-gameplay.md` | Core mechanics, controls, AI specs |
| `docs/prompts/08-file-structure.md` | Planned file organization |
| `test.sh` | Repository verification script |

## Conventions

### Documentation Style
- Use markdown with clear headers
- Include verification checkpoints at end of each doc file
- Cross-reference related docs (e.g., "See 01-gameplay.md")
- Keep line length reasonable for readability

### File Naming
- Use lowercase with hyphens for file names (e.g., `repo-onboarding.md`)
- Use descriptive names that indicate purpose
- JavaScript: camelCase for variables/functions, PascalCase for classes
- CSS: kebab-case for class names

### Code Style (When Implementation Begins)
- **JavaScript**: ES6+ syntax, vanilla JS (no frameworks)
- **Classes**: Use class syntax for game objects (Game, AI, Controls, etc.)
- **Constants**: UPPER_SNAKE_CASE in config.js
- **Comments**: Only when necessary to explain complex logic
- **Async**: Use async/await over raw Promises

### Git Workflow
- Work on feature branches
- Descriptive commit messages
- Update AI_REPO_GUIDE.md when adding commands or changing structure

## Common Gotchas

### Repository Status
- **No implementation files exist yet** - This is expected and correct
- Repository contains comprehensive documentation and planning only
- Implementation will follow the 10-phase plan in `docs/prompts/README.md`

### File Locations
- Documentation goes in `docs/prompts/`
- AI configurations go in `.github/agents/`
- Shell scripts remain in repository root

### Documentation References
- Game specs reference a 10-phase implementation plan
- Start with Phase 1 (core gameplay) before adding features
- Each phase builds on previous phases
- Phases 6-8 require external services (Supabase, Railway)

### Design Constraints
- Target 60fps on mobile devices
- First to 11 points wins a match
- 10 different power-up types
- 4 AI difficulty levels (Easy, Medium, Hard, Impossible)
- Support both portrait and landscape orientations

### External Dependencies (Future)
- **Supabase**: PostgreSQL database, requires free account and API keys
- **Railway**: Backend hosting, requires free account
- **GitHub Pages**: Frontend hosting (free with GitHub)

## Before Submitting Changes

Always verify:
1. [ ] Repository structure validated: `./test.sh`
2. [ ] AI_REPO_GUIDE.md updated if structure/commands changed
3. [ ] Changes align with specs in `docs/prompts/`
4. [ ] No build artifacts or temporary files committed
5. [ ] Tests pass locally: `npm test`
6. [ ] Lint passes: `npm run lint`
7. [ ] CI is green after push (check GitHub Actions)

## Quality Bar

- Don't guess APIs/behavior; cite repo files
- Refer to specifications in `docs/prompts/` for implementation details
- Run verification commands (prefer those in AI_REPO_GUIDE.md)
- If your changes affect commands/layout/conventions/troubleshooting: update AI_REPO_GUIDE.md (or state "AI_REPO_GUIDE.md: no changes required")

## Onboarding / Refresh Instructions

- If AI_REPO_GUIDE.md is missing/stale: follow `.github/prompts/repo-onboarding.md` and update AI_REPO_GUIDE.md in the same PR
- If this file (`.github/copilot-instructions.md`) is missing or clearly generic/stale: follow `.github/prompts/copilot-onboarding.md` to regenerate it *after* AI_REPO_GUIDE.md is accurate

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mikejmckinney) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
