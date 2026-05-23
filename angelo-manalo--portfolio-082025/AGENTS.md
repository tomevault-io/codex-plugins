# Copilot Instructions

## Project Structure & Architecture
This is a Next.js app using the `/src/app` directory structure. Key locations:
- **Main entry**: `src/app/page.tsx` 
- **Global layout**: `src/app/layout.tsx` for app-wide wrappers
- **Global styles**: `src/app/globals.css` with Tailwind CSS
- **Static assets**: `public/` directory, reference as `/file.svg`
- **Components**: Place UI components under `src/components/`

## Critical File Naming Convention
**ALWAYS use kebab-case** for ALL files and directories (e.g., `server-card.tsx`, `login-form.tsx`, `dashboard-layout.tsx`). Never introduce PascalCase or camelCase filenames. When touching legacy PascalCase files, rename them to kebab-case using PowerShell.

## ShadCN UI Workflow (Dual Approach)
This project uses both ShadCN UI MCP server and CLI:

1. **Plan first**: Use MCP server to get vetted patterns and component demos
2. **Generate components**: Run consolidated CLI commands to add missing components:
   ```bash
   npx shadcn@latest add button card input label badge skeleton tabs accordion dialog alert navigation-menu separator scroll-area avatar dropdown-menu toast
   ```
3. **Implement exactly**: Mirror MCP demo structure, variants, and accessibility attributes
4. **Extend via composition**: Create wrapper components instead of editing generated UI primitives

## Git & Development Workflows

### Branching (GitHub Flow)
- Always start from updated `main`: `git checkout main && git pull --rebase`
- Create feature branches: `git checkout -b feat/short-description`
- Use `git pull --rebase` instead of `git pull`
- Only use `feat:` prefix when merging PRs (not individual commits)

### Conventional Commits
Format: `<type>[optional scope]: <description>`
- **feat**: new feature (PR merges only)
- **fix**: bugfix  
- **refactor**: code change without bug fix or feature
- **chore**: dependencies, tooling
- **style**: formatting, whitespace
- **docs**: documentation only

### File Operations via PowerShell
**Deletions**: Use `Remove-Item -Path .\path\to\file.tsx` (never just clear contents)
**Renames**: Use `Rename-Item -Path .\old-name.tsx -NewName new-name.tsx` (enforces Git rename detection)

## Development Commands
- **Start dev**: `npm run dev` (auto-reloads on save)
- **Add new page**: Create `src/app/[route]/page.tsx`
- **Icons**: Use `lucide-react` for UI elements
- **Styling**: Tailwind utilities preferred over custom CSS

## Project Rules
- Use React functional components and hooks
- Keep all app code under `src/app/`
- No API routes exist by default (add in `src/app/api/` if needed)
- Place new routes as `src/app/[route-name]/page.tsx`
 
## Code Standards
- Use TypeScript for type safety
- Follow ESLint and Prettier configurations
- Maintain consistent code formatting
- Use semantic commit messages

---
> Source: [angelo-manalo/portfolio-082025](https://github.com/angelo-manalo/portfolio-082025) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-23 -->
