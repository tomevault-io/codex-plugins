# Copilot Instructions for multi-post-dashboard

## Project Overview
This is a client-side dashboard for generating and managing social media posts (YouTube, TikTok, Facebook, Instagram) with SEO optimization. The app is built with vanilla JavaScript, HTML, and CSS, and relies on localStorage for state and a remote API for advanced features.

## Architecture & Data Flow
- **Entry Points:**
  - `index.html`: Main dashboard (not included above, but inferred)
  - `folder.html`: Manages folders and linked social accounts
  - `create-post.html`: Generates SEO-optimized content for posts
- **State Management:**
  - Uses `localStorage` for folders, active folder/account, and user email
  - Folder/account data is loaded and rendered dynamically
- **API Integration:**
  - All remote calls use `apiBase` (Cloudflare Worker API)
  - Endpoints: `/get-accounts`, `/auth/{platform}`, `/generate-seo`
  - OAuth flows for social platforms are handled via redirects to Worker endpoints
- **UI Patterns:**
  - Responsive, mobile-first design (see `styles.css`)
  - Cards and tabs for navigation and content separation
  - Platform-specific content areas are shown/hidden based on active folder/account

## Developer Workflows
- **No build step required**; static files are directly served
- **Testing:**
  - Use `test-auth.js` to verify localStorage-based authentication
  - Manual browser testing recommended for OAuth and API flows
- **Debugging:**
  - Use browser DevTools to inspect localStorage, network requests, and UI state
  - Console logs are used for error reporting and debugging

## Project-Specific Conventions
- **Platform logic:**
  - Platform names are lowercased (`facebook`, `instagram`, `tiktok`, `youtube`)
  - Platform-specific UI elements are toggled via `.tab` and `.hidden` classes
- **SEO Generation:**
  - `create-post.html` calls `/generate-seo` and populates fields for each platform
  - Copy buttons use `navigator.clipboard` and open platform upload pages
- **Account Linking:**
  - OAuth flows are triggered from `folder.html` and handled by Worker API
  - Folder/account context is passed via `folder_id` in query params/state

## Integration Points
- **Cloudflare Worker API:**
  - Update `apiBase` in HTML/JS files if endpoint changes
  - All account and SEO features depend on this API
- **YouTube OAuth:**
  - Configure `YOUTUBE_CLIENT_ID` and `YOUTUBE_REDIRECT_URI` in `youtube-auth.js`

## Key Files
- `app.js`: Main dashboard logic, tab switching, content generation
- `folder.html`: Folder/account management, OAuth triggers
- `create-post.html`: SEO content generation, platform-specific copy actions
- `styles.css`: UI layout and theming
- `test-auth.js`: Local authentication test
- `youtube-auth.js`: YouTube OAuth flow

## Example Patterns
- To add a new platform, update folder/account logic and UI tab toggling in `app.js` and HTML files
- To change API endpoints, update `apiBase` in all relevant files
- To debug OAuth, check localStorage for `activeFolderId` and inspect redirect URLs

---
**For AI agents:**
- Always check localStorage for current folder/account context
- Use the remote API for all account and SEO operations
- UI changes should follow the card/tab pattern and use platform-specific classes
- Reference the above files for canonical implementations of each feature

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mgt581) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
