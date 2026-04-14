# Copilot Instructions for AI Coding Agents

## Project Overview
This is a browser extension for Twitter/X that displays account-based location information next to usernames. It works on both Firefox and Chrome, using content scripts and page script injection to fetch and display location data from Twitter's GraphQL API.

## Architecture & Key Components
- **src/content.js**: Main content script. Detects usernames, communicates with the injected page script, and updates the UI with location info.
- **src/pageScript.js**: Injected into the page context. Handles API requests to Twitter's GraphQL endpoint using the user's authentication/cookies.
- **src/cacheManager.js**: Manages caching of location data in IndexedDB, with a 30-day expiry.
- **src/background.js**: Handles extension background tasks (minimal usage).
- **src/popup.js / src/popup.html**: UI for extension popup (not core to location logic).
- **manifest.*.json**: Browser-specific extension manifests. Use `manifest.json` for general logic, and override with `manifest.chrome.json` or `manifest.firefox.json` as needed.

## Data Flow
1. **Detection**: `src/content.js` scans Twitter/X pages for usernames.
2. **Request**: For each username, it triggers a custom event handled by `src/pageScript.js`.
3. **API Call**: `src/pageScript.js` queries Twitter's GraphQL API for location info.
4. **Caching**: Results are cached via `src/cacheManager.js` in IndexedDB.
5. **Display**: Location info is injected next to usernames in the UI.

## Developer Workflows
-- **Build**: Use the `Makefile` for packaging the extension. The `Makefile` now packages files under `src/` (e.g. `src/content.js`, `src/pageScript.js`). Example: `make firefox` or `make chrome` creates a distributable ZIP in `dist/`.
- **Manual Install**: Load the extension in Firefox via `about:debugging` or in Chrome via `chrome://extensions` (see README for details).
- **Debugging**: Use browser console logs in Twitter/X pages. Check for errors in content/page scripts. IndexedDB can be inspected for cache issues.
- **Testing**: No automated test suite; manual testing in browsers is required.

## Project-Specific Patterns
- **API Integration**: All Twitter API calls are made from the injected page script (`src/pageScript.js`) to avoid CORS/auth issues.
- **Caching**: Location data is cached for 30 days to minimize API calls and avoid rate limits.
- **Rate Limiting**: Extension warns users if Twitter's API rate limit (50 requests/15 min) is reached.
- **Manifest Handling**: Always check which manifest file is active for browser-specific logic.

## External Dependencies
- No third-party libraries; all logic is custom JavaScript.
- Relies on Twitter/X's public GraphQL API and browser IndexedDB.

## Example Patterns
- **Custom Event Communication**: See how `content.js` and `pageScript.js` use custom DOM events for cross-context messaging.
- **IndexedDB Usage**: `cacheManager.js` provides a simple pattern for storing and expiring cached data.

## Key Files
-- `src/content.js`, `src/pageScript.js`, `src/cacheManager.js`, `manifest.*.json`, `Makefile`, `README.md`

---

For questions or unclear patterns, review the README or open an issue. Please update this file if new workflows or architectural changes are introduced.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mamantoha) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
