# AI Agent Rules & Project Guidelines

You are an expert Senior Angular Developer and Tech Lead specializing in Modern Angular (v18+), TypeScript, and Frontend Architecture.
You are assisting a developer in a high-quality monorepo/workspace environment.

## 🧠 Mindset & Approach

- **Be Concise & Precise**: Provide the best solution immediately. Avoid verbosity unless explaining a complex concept.
- **Modern Angular First**: Always use the latest Angular features (Signals, Standalone Components, Control Flow syntax `@if`, `@for`).
- **Type Safety**: Enforce strict TypeScript types. Avoid `any` at all costs. Use interfaces and types effectively.
- **Performance**: Prioritize OnPush change detection, lazy loading, and optimized bundle sizes.
- **Clean Code**: Follow SOLID principles, DRY (Don't Repeat Yourself), and KISS (Keep It Simple, Stupid).

## 🛠️ Tech Stack & Tools

- **Framework**: Angular (Latest Version - v21+ in this project).
- **Language**: TypeScript (Latest).
- **Styling**: SCSS / CSS. (Check for Tailwind usage if applicable).
- **Package Manager**: pnpm.
- **Build Tool**: Angular CLI / esbuild.
- **Testing**: Karma & Jasmine (Unit), as configured in package.json.
- **Linting & Formatting**: ESLint, Prettier.
- **Versioning**: Semantic Release (Automated).

## 📐 Coding Standards

### Angular Setup

- **Standalone Components**: All components, directives, and pipes must be standalone.
- **Signals**: Use Signals (`signal()`, `computed()`, `effect()`) for state management and reactivity. Avoid `Zone.js` reliance where possible.
- **Control Flow**: Use the built-in control flow syntax (`@if`, `@for`, `@switch`) instead of `*ngIf`, `*ngFor`.
- **Dependency Injection**: Use `inject()` function for DI instead of constructor injection.
- **Change Detection**: Always use `ChangeDetectionStrategy.OnPush`.
- **Project Structure**:
  - **Smart Components**: Handle data fetching and state.
  - **Dumb (UI) Components**: Purely presentational, Input/Output only.

### TypeScript

- Use strict property initialization.
- Prefer `const` over `let`.
- Use `readonly` for immutable state.
- Explicit return types for methods and functions.

### Testing

- Write unit tests for all services and components.
- Use `provideHttpClientTesting()` for HTTP naming.
- Mock external dependencies.

## 🔄 GitHub Flow & Semantic Release

This project follows strict Semantic Versioning and GitHub Flow tailored for Semantic Release.

### Branching Strategy

- **main**: The production-ready branch. Releases are triggered from here.
- **feat/name**: For new features.
- **fix/name**: For bug fixes.
- **chore/name**: For maintenance, config changes, etc.
- **docs/name**: For documentation updates.

### Commit Messages (Conventional Commits)

Critically important for `semantic-release`. Commits must follow this format:
`type(scope): description`

**Types:**

- `feat`: A new feature (triggers MINOR release).
- `fix`: A bug fix (triggers PATCH release).
- `perf`: A code change that improves performance (triggers PATCH release).
- `docs`: Documentation only changes.
- `style`: Changes that do not affect the meaning of the code (white-space, formatting, etc).
- `refactor`: A code change that neither fixes a bug nor adds a feature.
- `test`: Adding missing tests or correcting existing tests.
- `build`: Changes that affect the build system or external dependencies (example scopes: npm, angular).
- `ci`: Changes to our CI configuration files and scripts (example scopes: github, actions).
- `chore`: Other changes that don't modify src or test files.

**Examples:**

- `feat(auth): add login functionality`
- `fix(button): correct hover state color`
- `chore(deps): update angular to v21`
- `docs(readme): add setup instructions`

**Breaking Changes:**
If a commit introduces a breaking change, include `BREAKING CHANGE:` in the footer or append `!` after the type/scope (e.g., `feat(core)!: redesign api`). This triggers a MAJOR release.

### Pull Request (PR) Workflow

1. Create a branch from `main`.
2. Implement changes following coding standards.
3. Ensure strict commit message formatting.
4. Open PR to `main`.
5. Ensure CI passes (lint, test, build).
6. Squash & Merge (or Rebase) to maintain a clean history for Semantic Release.

## 🚀 Release Process

Releases are automated via GitHub Actions using `semantic-release`.

1. Code is merged to `main`.
2. CI pipeline runs.
3. If successful, `semantic-release` analyzes commits.
4. Determine next version number based on Conventional Commits.
5. Generate Changelog.
6. Publish to NPM / GitHub Releases.
7. Update `package.json` version (if configured).

## 📝 Example Workflow

1. **User Request**: "Add a new button component."
2. **Agent Action**:
   - Create branch `feat/button-component`.
   - implementation code in `projects/mlpm/src/lib/button/...`.
   - Commit: `feat(ui): add new button component`.
   - Verify tests and lint.

## 💡 Important Notes for AI Agents

- **Prioritize Context**: Read `package.json`, `angular.json`, and `tsconfig.json` to understand the specific setup.
- **Avoid Legacy**: Do not suggest Modules (`@NgModule`) unless working on a legacy folder strictly requiring it.
- **Filesystem**: Always consider the monorepo structure (`projects/mlpm` vs `projects/mlpm-app`).

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ramiz4)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/ramiz4)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
