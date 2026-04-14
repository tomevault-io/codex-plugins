# Copilot Coding Agent Instructions

## Project Summary

**Blocks Randomizer** is a WordPress Gutenberg block plugin that displays randomly selected child blocks from a container on the front-end. It is server-side rendered for cache-friendly randomization.

- **Type**: WordPress plugin (Gutenberg block)
- **Languages**: JavaScript (ES6+), PHP 7.4+, SCSS
- **Framework**: `@wordpress/scripts` build toolchain
- **WordPress**: Requires 6.7+, tested up to 6.9
- **Size**: Small (~15 source files)

## Build Commands

**Always run these in order for a clean build:**

```bash
npm ci                  # Install dependencies (use ci, not install)
npm run build           # Production build (~15s)
```

**Other commands:**

| Command               | Purpose                              | Notes                           |
|-----------------------|--------------------------------------|---------------------------------|
| `npm start`           | Development build with watch mode    | Use for active development      |
| `npm run build`       | Production build                     | Required before plugin-zip      |
| `npm run plugin-zip`  | Create distributable ZIP             | Run build first                 |
| `npm run format`      | Format code (WordPress standards)    | Fixes many lint issues          |
| `npm run lint:js`     | Lint JavaScript (ESLint)             | Must pass with no errors        |
| `npm run lint:css`    | Lint SCSS (Stylelint)                | Must pass with no errors        |

## Important Build Notes

1. **Always use `npm ci`** instead of `npm install` for reproducible builds.
2. **Node.js 20** is required (see `.nvmrc`).
3. **Build before zipping**: The `plugin-zip` command requires a successful `build` first.
4. **Linting must pass**: Both `npm run lint:js` and `npm run lint:css` must complete with no errors. Fix any linting issues in your changes.
5. **The build directory is gitignored** and generated fresh on each build.

## Project Layout

```
/                           # Repository root
├── blocks-randomizer.php   # Main plugin file (block registration)
├── package.json            # npm scripts and dependencies
├── .nvmrc                  # Node.js version (20)
├── README.md               # User documentation
├── readme.txt              # WordPress.org plugin readme
├── CLAUDE.md               # Claude-specific guidance
├── src/
│   └── blocks-randomizer/  # Block source files
│       ├── block.json      # Block metadata and attributes
│       ├── edit.js         # Editor component (React)
│       ├── save.js         # Save component (stores inner blocks)
│       ├── render.php      # Server-side randomization logic
│       ├── index.js        # Block registration
│       ├── view.js         # Front-end script (empty, SSR used)
│       ├── editor.scss     # Editor-only styles
│       └── style.scss      # Front-end + editor styles
├── build/                  # Generated (gitignored)
│   ├── blocks-manifest.php # Auto-generated block manifest
│   └── blocks-randomizer/  # Compiled block assets
├── .github/
│   └── workflows/
│       ├── claude.yml              # Claude Code workflow
│       └── claude-code-review.yml  # Claude code review workflow
└── .wp-org/                # WordPress.org assets (banners, icons)
```

## Key Files for Changes

| When modifying...           | Edit these files                                |
|-----------------------------|-------------------------------------------------|
| Block behavior (front-end)  | `src/blocks-randomizer/render.php`              |
| Block editor UI             | `src/blocks-randomizer/edit.js`                 |
| Block attributes            | `src/blocks-randomizer/block.json` and `edit.js`|
| Styles                      | `src/blocks-randomizer/editor.scss` or `style.scss` |
| Block registration          | `src/blocks-randomizer/index.js`                |
| Plugin metadata             | `blocks-randomizer.php` and `readme.txt`        |

## Validation Checklist

Before submitting changes:

1. **Build succeeds**: `npm run build` completes without errors
2. **Format code**: `npm run format` (run before linting)
3. **Check linting**: `npm run lint:js && npm run lint:css` must pass with no errors
4. **Test in WordPress**: Plugin requires WordPress 6.7+ environment

## No Test Suite

This repository does not have automated tests configured. Manual testing in a WordPress environment is required.

## Code Style

- Follow [WordPress Coding Standards](https://developer.wordpress.org/coding-standards/)
- Use tabs for indentation
- PHP files use WordPress conventions (`phpcs:ignore` comments are intentional in `render.php`)
- Use `@wordpress/i18n` for translatable strings

## Dependencies

- Single dev dependency: `@wordpress/scripts` (includes webpack, eslint, stylelint, prettier)
- WordPress packages (`@wordpress/blocks`, `@wordpress/i18n`, etc.) are externals resolved at runtime

## Workflow Notes

- No CI/CD builds or tests are configured beyond Claude workflows
- Dependabot monitors npm packages weekly
- GitHub Actions workflows are for Claude-based code review only

Trust these instructions. Only search the codebase if information here is incomplete or incorrect.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/slaFFik) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
