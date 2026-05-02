# Copilot Instructions for AutoPlan

AutoPlan is a plugin for Super Productivity that automatically schedules tasks based on urgency. This document provides guidance for working with this codebase.

## Project Overview

- **Type**: Super Productivity plugin (browser-based)
- **Language**: JavaScript (ES Modules)
- **Testing**: Vitest
- **Build**: Custom Node.js build script (`build.js`)

## Project Structure

- `src/core.js` - Core library with testable, pure functions (no PluginAPI dependencies)
- `src/plugin-template.js` - Plugin-specific code that uses PluginAPI
- `src/index.html` - Source HTML for the plugin settings UI (edit this file, not plugin/index.html)
- `plugin/` - Built plugin files (plugin.js, manifest.json, index.html) - DO NOT EDIT DIRECTLY
- `tests/` - Vitest test files
- `ai/DESIGN.md` - Original design document

## Development Commands

```bash
npm install          # Install dependencies
npm run build        # Generate plugin/plugin.js and minify plugin/index.html
npm run build:zip    # Create release zip
npm test             # Run tests with Vitest
npm run test:watch   # Run tests in watch mode
```

## Build Process

The build script (`build.js`) performs two tasks:

1. **Bundles JavaScript**: Combines `src/core.js` and `src/plugin-template.js` into `plugin/plugin.js`
2. **Minifies HTML**: Minifies `src/index.html` to `plugin/index.html` (reduces size by ~20%)

**IMPORTANT**: Always edit `src/index.html`, never edit `plugin/index.html` directly. The plugin manifest has a 100KB size limit, so minification is required.

## Code Style and Conventions

- Use ES Modules (`export`/`import`) syntax
- Core logic should be in `src/core.js` to remain testable without PluginAPI
- Plugin-specific code using PluginAPI belongs in `src/plugin-template.js`
- Export functions and objects that need testing
- Use JSDoc-style comments for function documentation

## Testing Guidelines

- Tests are in the `tests/` directory with `.test.js` suffix
- Use Vitest's `describe`, `it`, and `expect`
- Create helper functions for test data (e.g., `createTask()`)
- Test edge cases like null/undefined inputs, empty arrays, and boundary conditions

## Key Modules

### PriorityCalculator
Calculates task urgency based on tags, projects, duration, and age.

### TaskSplitter
Splits large tasks into time blocks with Roman numeral suffixes (e.g., "Task <I>", "Task <II>").

### AutoPlanner
Main scheduling algorithm that assigns time blocks to tasks based on urgency. Supports multiple time maps for per-project scheduling windows.

### TaskMerger
Handles merging split tasks back into their original form.

## Important Notes

- The plugin runs in a sandboxed iframe environment
- `PluginAPI` is only available at runtime in Super Productivity
- Keep core logic separated from plugin-specific code for testability
- Roman numerals are used for split task numbering (I, II, III, etc.)
- Time estimates are stored in milliseconds in Super Productivity
- The plugin manifest has a 100KB size limit - always run `npm run build` to minify HTML

---
> Source: [00sapo/sp-autoplan](https://github.com/00sapo/sp-autoplan) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-02 -->
