# Silo AI Coding Instructions

## Project Overview

Silo is an Electron-based macOS desktop wrapper for Google Workspace (Gmail, Calendar, Drive, Keep, Tasks, Contacts) and Gemini. It uses **BrowserViews** (not BrowserWindows or webview tags) to create persistent, side-by-side views that feel native.

**Core Architecture**: Single main window + sidebar BrowserView + multiple content BrowserViews (one per Google app). Only one content view is visible at a time alongside the persistent sidebar.

## Key Architectural Patterns

### BrowserView Management ([src/main/main.js](src/main/main.js))

- **BrowserViews over tabs**: Each Google app lives in its own `BrowserView`, not a tab or separate window. Views persist in memory even when hidden, maintaining state.
- **View switching**: `showView(name)` removes all views from the window, then re-adds sidebar + the target content view. Uses `setBounds()` for layout, not CSS.
- **Fixed sidebar**: The sidebar is always 60px wide, positioned at x:0. Content views start at x:60 and fill remaining width.
- **View registry**: All views stored in the `views` object keyed by name (mail, calendar, drive, etc.). The `currentView` global tracks which content view is active.

### Security & URL Handling

- **Internal domain whitelist**: `INTERNAL_DOMAINS` array defines allowed Google domains. Any external link opens in the default browser via `shell.openExternal()`.
- **Window open handler**: `setWindowOpenHandler()` checks `isInternalUrl()` for all new window requests. Only internal URLs get `{ action: "allow" }`.
- **Navigation guard**: `will-navigate` event prevents navigation to external domains and redirects them externally.

### IPC Communication

- **Preload scripts segregated**:
  - [preload.js](src/preload/preload.js) for content views - exposes `notify()` and `unreadCount()` to Gmail
  - [sidebar-preload.js](src/preload/sidebar-preload.js) for sidebar - exposes `switch()` and `onActiveChange()`
- **Badge updates**: Gmail view sends `unread-count` IPC message, main process updates macOS dock badge
  - Implementation may use Google's existing APIs or DOM scraping when APIs unavailable
  - Check Gmail's web interface for exposed notification data before implementing custom scraping
- **View switching**: Sidebar clicks trigger `sidebar-switch` IPC, main process calls `showView()`

### Menu & Keyboard Shortcuts

- **Cmd+1 through Cmd+7**: Switch between the 7 Google apps
- **Cmd+N**: Compose new email (opens in separate `BrowserWindow`)
- **Cmd+,**: Open settings view
- **Template icons**: Menu icons use `nativeImage.setTemplateImage(true)` for native macOS appearance

## Development Workflow

### Running the App

```bash
npm start              # Default: Gmail wrapper
npm run start:gmail    # Explicit Gmail
npm run start:calendar # Start with Calendar active
```

The `WRAPPER_APP` environment variable determines the initial view, but this is legacy - the app now shows all services simultaneously via sidebar switching.

### Building for macOS

```bash
npm run build
```

Uses `electron-builder` with custom `build` config in [package.json](package.json). Output goes to `dist/`. Currently configured for Mac only with:
- `public.app-category.productivity` category
- `mailto:` protocol handler registration
- Code signing and notarization disabled (`sign: false`, `notarize: false`)

### Project Structure

```
src/
  main/main.js          # Main process - window/view management, menus, IPC
  preload/
    preload.js          # Preload for content views (Google apps)
    sidebar-preload.js  # Preload for sidebar
  renderer/
    sidebar.html        # 60px sidebar UI with Material icons + custom SVGs
    settings.html       # Settings view (currently placeholder)
    img/                # Custom icons (drive.png, gemini.png) as CSS masks
assets/
  icon.icns            # App icon
  menu/*.png           # Template images for native menus
```

## Code Style & Conventions

- **No frameworks**: Vanilla JavaScript, plain HTML/CSS. No React, Vue, or similar.
- **CommonJS**: Uses `require()`, not ES modules. `"type": "commonjs"` in package.json.
- **Inline styles**: Renderer HTML files have `<style>` blocks, no separate CSS files.
- **Material Symbols**: Sidebar uses Google's Material Symbols font (outlined style). Custom icons (Drive, Gemini) are PNG masks applied via CSS.
- **Template literals for file paths**: `file://${path.join(__dirname, "../renderer/settings.html")}`

## Testing Approach

This is a solo project with **manual testing only**. No automated test suite exists. When making changes:
- Test all 7 Google app views (mail, calendar, drive, gemini, keep, tasks, contacts)
- Verify keyboard shortcuts (Cmd+1-7, Cmd+N, Cmd+,, Cmd+R)
- Check both light and dark mode appearance
- Test external link handling (should open in default browser)
- Verify sidebar active state syncs correctly with view switching

## Custom Icon Usage

Drive and Gemini icons are custom PNGs in [src/renderer/img/](src/renderer/img/). Applied as CSS masks:

```css
.drive-icon {
  mask: url("./img/drive.png") no-repeat center / contain;
  -webkit-mask: url("./img/drive.png") no-repeat center / contain;
  background-color: #4a4a4a; /* changes with active state */
}
```

**Why masks?** Allows dynamic color changes without multiple image files. Active state changes `background-color` to black/white depending on theme.

**Icon creation**: Custom icons are modeled after Material Design guidelines. Use Material Design as reference when Google doesn't provide an official icon for a service (Drive and Gemini currently use custom icons).

## Adding a New Google App View

1. Add URL to `VIEW_URLS` object in [main.js](src/main/main.js)
2. Add domain to `INTERNAL_DOMAINS` array
3. Add sidebar item to [sidebar.html](src/renderer/sidebar.html) with unique id
4. Add menu item to "Switch To" submenu with Cmd+N accelerator
5. Create view in `createMainWindow()` via `views[key] = createContentView(key)`

**Note**: All views are created at startup and persist in memory. Lazy loading is not implemented.

## Common Pitfalls

- **BrowserView bounds**: Must call `setBounds()` explicitly after window resize. CSS won't affect BrowserView positioning.
- **View z-order**: The order you call `addBrowserView()` matters. Sidebar must be added last to stay on top.
- **Preload script paths**: Use `path.join(__dirname, "../preload/...")` - relative paths from main.js location, not project root.
- **External links**: Always check if you need to update `INTERNAL_DOMAINS` when adding new Google services.

## Planned Features & Roadmap

### Multi-Profile Support (Next Priority)

The app currently supports only a single Google account. The next major feature is **profile-based sessions** that allow users to:
- Switch between multiple Google accounts
- Maintain isolated session environments per profile
- Store profile-specific settings and state

This will require:
- Session partitioning for BrowserViews (`partition: 'persist:profile-name'` in webPreferences)
- Profile management UI (likely in settings view)
- Persistent storage for profile configurations (consider `electron-store`)

### TypeScript Migration

Considering migration from JavaScript to TypeScript for better type safety and developer experience. This would be a significant refactor affecting all `.js` files.

## Git Workflow

Silo uses a **simplified Git Flow** approach:

### Branch Structure
- **`main`** - Production-ready, stable code
- **`develop`** - Integration branch for features
- **`feature/*`** - New features (e.g., `feature/native-notifications`)
- **`bugfix/*`** - Bug fixes for develop
- **`hotfix/*`** - Urgent production fixes
- **`release/*`** - Release preparation

### Workflow Summary
1. Create feature branches from `develop`
2. Develop and commit with conventional commit messages
3. Merge back to `develop` when complete
4. Create release branches from `develop` when ready
5. Merge releases to both `main` and `develop`
6. Tag releases on `main` (e.g., `v1.1.0`)

### Commit Convention
Follow conventional commits: `type(scope): subject`

**Types**: feat, fix, docs, style, refactor, test, chore, perf

**Example**: `feat(profiles): add custom avatar upload functionality`

See [CONTRIBUTING.md](../CONTRIBUTING.md) for detailed workflow documentation.

### Version Management
- Follow Semantic Versioning (MAJOR.MINOR.PATCH)
- Update version in package.json before releases
- Maintain [CHANGELOG.md](../CHANGELOG.md) with all changes
- Current version: 1.0.0

## Future Expansion Areas

- Windows/Linux support not tested (BrowserView behavior may differ)
- No autoupdate mechanism configured in electron-builder

---
> Source: [mattlinebarger/silo](https://github.com/mattlinebarger/silo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-30 -->
