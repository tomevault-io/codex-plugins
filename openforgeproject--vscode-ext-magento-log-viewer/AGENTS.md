# Copilot Instructions for Magento Log Viewer Extension

## Core Principles

**Maintainability is the primary goal.** Clean, well-structured code over quick solutions. When maintainability and performance conflict, prefer maintainability unless the performance impact is measurable and user-visible.

**Performance is a feature** - every change must consider user experience impact. Optimize where profiling shows a real bottleneck, not preemptively.

## License Compliance

- **License:** GPL-3.0-or-later
- **Copyright:** Copyright (c) 2024-present OpenForge Project Contributors
- **Headers:** All new source files must include the SPDX identifier:
  ```typescript
  /**
   * SPDX-License-Identifier: GPL-3.0-or-later
   * Copyright (c) 2024-present OpenForge Project Contributors
   */
  ```
- When modifying an existing file that does not yet contain the SPDX header, add the header at the top of the file as part of the same change.

## Architecture Overview

VS Code extension for intelligent Magento log file viewing and management:

- **`src/extension.ts`** - Entry point with async initialization
- **`src/logViewer.ts`** - Log and Report TreeDataProviders, theme grouping logic
- **`src/helpers.ts`** - Utilities, caching system, auto-cleanup, **real-time tailing engine**

When a file named in the Architecture Overview must be split, retain the original filename as a re-export barrel (e.g., `helpers.ts` re-exports from `helpers/caching.ts`, `helpers/tailing.ts`) so external imports remain unchanged.
- **`src/updateNotifier.ts`** - Version update notifications

## Major Features (v1.25+)

### Real-Time Log Tailing

- Live monitoring with `TailingManager`
- Low-overhead incremental updates (only reads new bytes)
- Visual indicators (📡) and auto-scroll
- Configurable update intervals (100ms - 2s)

### Auto-Cleanup System

- Time-based cleanup: `autoCleanupMaxAge` ("30min", "2h", "7d", "2w", "3M")
- Periodic cleanup: `periodicCleanupInterval` (5min to 24h)
- Functions: `autoCleanupOldLogFiles()`, `startPeriodicCleanup()`, `stopPeriodicCleanup()`

### Theme-Aware Log Grouping

- `groupByMessage` setting reduces duplicate entries
- Detects "Broken reference" patterns and theme issues
- Auto expansion/collapse (see `themeGrouping.test.ts`)

## Performance Guidelines

When trade-offs are required, apply guidelines in this order: (1) Startup, (2) Memory, (3) File I/O, (4) UI, (5) Background.

- **Startup**: Never block VS Code startup
- **Memory**: Monitor usage, implement cleanup, avoid leaks
- **UI**: Use async/await, throttle updates, prevent blocking
- **File I/O**: Cache reads, stream large files, batch operations
- **Background**: Offload CPU-intensive work using Node.js `worker_threads` or by deferring with `setImmediate`; do not block the extension host event loop.

## Code Organization

- Split large functions (max 25 lines); exception: orchestration functions that coordinate multiple steps (e.g., the extension activation sequence) may exceed this limit — extract each step into a named helper instead
- Split files when >450 lines
- Single responsibility per function/class
- Use self-documenting names instead of inline comments. JSDoc is permitted for exported public APIs; inline comments are not.
- Testable units
- Strict TypeScript, avoid `any`

## Key Patterns

### Workspace-Scoped Configuration

`vscode.workspace.getConfiguration('magentoLogViewer', workspaceUri)` - per-workspace settings

### Asynchronous Initialization

Extension uses 500ms delay for file operations to avoid blocking VS Code startup

### Intelligent Caching

Multi-layered in `helpers.ts`:

- Line count: `lineCountCache` with timestamp invalidation
- File content: `fileContentCache` with size limits
- Reports: `reportCache` for parsed JSON
- Dynamic config: `getCacheConfig()` adapts to memory/settings

### Context-Driven UI

Commands shown via `when` clauses:

- `magentoLogViewer.hasMagentoRoot`
- `magentoLogViewer.hasActiveSearch/hasActiveSearchReports`
- `magentoLogViewer.hasLogFiles`
- `magentoLogViewer.autoCleanupEnabled/periodicCleanupEnabled`
- `magentoLogViewer.hasTailedFiles/tailingActive`

### Memory Management

- Dispose pattern: cleanup disposables array
- Auto optimization: `optimizeCacheSize()`
- Proper disposal in `deactivate()`
- `TailingManager` resource cleanup

### Error Handling

- User-friendly messages with action buttons
- Graceful degradation with fallbacks
- Console.error for debugging
- All file system operations that depend on `magentoRoot` must validate the path exists and is a directory before proceeding; on failure, surface a `vscode.window.showErrorMessage` with an "Open Settings" action button and return early without throwing.

## Project Conventions

### File System

- Logs: `{magentoRoot}/var/log/*.log`
- Reports: `{magentoRoot}/var/report/` (recursive)
- Icons: Color-coded by log level

### Search

- Real-time filtering via `quickPick.onDidChangeValue`
- Regex support with `searchUseRegex`
- Case sensitivity via `searchCaseSensitive`

### Extension Flow

1. Activate on `onStartupFinished`
2. Check `isMagentoProject` setting
3. Validate `magentoRoot` path
4. Create providers, activate watchers

## Integration

### VS Code APIs

TreeDataProvider, FileSystemWatcher, QuickPick, Workspace Configuration, Status Bar

### Dependencies

Webpack (bundles to `dist/extension.js`), Node.js fs, VS Code Test Framework

### Performance

Lazy loading, throttled updates, smart caching, async file ops, batch operations, stream processing, debounced events

## Development

### Build

```bash
npm run compile  # Development build
npm run watch    # Watch mode
npm run package  # Production build
npm run test     # Run tests
```

### Testing

Tests in `src/test/` - one file per component:

- `extension.test.ts`, `caching.test.ts`, `search.test.ts`
- `autoCleanup.test.ts`, `themeGrouping.test.ts`
- Component tests: `cacheConfiguration.test.ts`, `logReader.test.ts`, `reportReader.test.ts`

Organization: descriptive names, mock dependencies, test edge cases

## Release

- never Update [CHANGELOG.md](../CHANGELOG.md) this will be done by the release manager when the release is cut.

## Git Conventions

- **Conventional Commits**: All Pull Requests titles must follow the Conventional Commits specification.
  - Structure: `type(scope): description`
  - Types: `feat`, `fix`, `docs`, `style`, `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`
  - Examples:
    - `feat: add real-time log tailing`
    - `fix: resolve crash on startup`
    - `docs: update README with new features`
- **Scopes**: Use file names or feature names (e.g., `logViewer`, `search`, `deps`).
- **Commits inside PRs**: Can be unstructured, as PRs should be squashed upon merge. The PR title will become the commit message on `main`.

---
> Source: [OpenForgeProject/vscode-ext-magento-log-viewer](https://github.com/OpenForgeProject/vscode-ext-magento-log-viewer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-16 -->
