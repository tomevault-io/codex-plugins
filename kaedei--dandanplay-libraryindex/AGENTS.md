# DanDanPlay Library Index - AI Coding Agent Instructions

## Project Overview
This is the web frontend for DanDanPlay's remote access feature, serving as a media library browser and video player interface. The HTML files are **server-side rendered** by NancyFx using the Super Simple View Engine (SSVE) - they are NOT static HTML files.

## Critical Architecture Concepts

### Server-Side Template Engine (SSVE)
All `.html` files are **templates processed server-side** before being sent to the browser:
- `@Model.PropertyName` - Access view model properties
- `@Each.Collection` / `@EndEach` - Loop through collections
- `@Current.PropertyName` - Access current loop item
- `@If.Condition` / `@IfNot.Condition` / `@EndIf` - Conditional rendering
- `@Partial['style.sshtml']` - Include partial templates
- `@!Current` - Output without HTML encoding (use with caution)

**Never modify or remove these template directives** - they are replaced at runtime by the C# backend.

### Page Architecture & Data Flow

**index.html** (Home page)
- Entry point accessed via `http://ip/`
- Displays media library navigation buttons
- Shows local and remote folder lists via `@Each.LocalFolderNames` / `@Each.RemoteFolderNames`

**bangumi.html** (Anime series list)
- Grid view of anime series with poster images
- Data loaded via AJAX from `@Model.BangumiListApi` endpoint
- Dynamic search and quick navigation features
- Opens modal to display episode list

**filelist.html** (Folder file browser)
- Lists video files in a selected folder (local or remote)
- Data loaded via AJAX from `@Model.FileListApi` endpoint
- Query params: `?type=local&name=FolderName` or `?type=remote&name=FolderName&path=SubPath`

**video.html** (Video player)
- Accessed via `http://ip/web/{video_hash}`
- Integrates DPlayer for video playback with danmaku (bullet comments)
- Uses SubtitlesOctopus for ASS/SSA subtitle rendering
- Optional VTT subtitle track via `?subtitle=filename`
- Episode list navigation in sidebar

**login.html** (Authentication)
- Shows when web authentication is enabled
- Anonymous users redirected here automatically

### Key File Structure

**style.sshtml** - Shared CSS partial included in all pages via `@Partial['style.sshtml']`
- Contains page-specific styles organized by sections
- Supports conditional custom backgrounds via `@If.UseCustomBackground`

**Third-party dependencies:**
- DPlayer 1.27.0 - Video player with danmaku support
- SubtitlesOctopus - Advanced subtitle renderer for ASS/SSA formats
- Bootstrap 5.3.3 (bangumi/filelist) and 4.0.0 (video) - UI framework
- Font Awesome 6.5.2 - Icons
- jQuery 3.7.1/3.2.1 - DOM manipulation
- lazysizes 5.3.1 - Lazy loading images

## Development Workflow

### Testing Changes Locally
1. Install DanDanPlay Windows/UWP client (latest version)
2. Enable "Remote Access" feature in app settings, enable developer tools
3. Clone this repo and copy files to DanDanPlay's `web` folder (typically in installation directory)
4. Access `http://127.0.0.1` or local IP shown in DanDanPlay settings
5. Edit HTML files and refresh browser - changes appear immediately
6. If unable to access, temporarily disable Windows Firewall

### Making Changes
- **HTML structure**: Safe to modify, but preserve all `@Model.*`, `@Each.*`, `@Current.*` directives
- **CSS**: Edit `style.sshtml` or page-specific `<style>` blocks
- **JavaScript**: Inline scripts in HTML files or external files in `js/` folder
- **Assets**: CSS/images go in `css/`, JavaScript in `js/`, web fonts in `webfonts/`

## Project-Specific Conventions

### Bootstrap Version Inconsistency
**Important**: Different pages use different Bootstrap versions:
- `bangumi.html`, `filelist.html`, `login.html`, `index.html` → Bootstrap 5.3.3
- `video.html` → Bootstrap 4.0.0 (legacy, likely for DPlayer compatibility)

**Always verify which Bootstrap version a page uses before adding/modifying components.**

### AJAX API Patterns
Pages use jQuery AJAX to load data from server endpoints:
```javascript
// Pattern: API URL from server-rendered @Model property
var apiUrl = "@Model.BangumiListApi";
$.ajax({
    url: apiUrl,
    method: 'GET',
    success: function(response) {
        // Response is JSON array, render dynamically
    }
});
```

### Lazy Loading Images
Use `lazysizes.js` for performance:
```html
<img class="lazyload" data-src="actual-image-url.jpg" src="placeholder.gif">
```

### Video Player Integration
DPlayer initialization must include:
- `crossorigin="use-credentials"` attribute on video element
- Danmaku API endpoint set to `'dplayer/'` (relative path)
- SubtitlesOctopus initialized separately for ASS subtitles

### Authentication State
Check authentication using `@IfNot.IsAnonymous` / `@If.IsAnonymous`:
```html
@IfNot.IsAnonymous
当前用户: @Model.UserName
<a href="/web1/logout">退出登录</a>
@EndIf
```

## Common Pitfalls

1. **Don't break SSVE syntax** - Malformed `@Model` directives cause server-side rendering errors
2. **URL encoding** - Use `@!Current` for pre-encoded URLs, avoid double-encoding in `href` attributes
3. **Bootstrap version mismatch** - Don't mix Bootstrap 4 and 5 class names on the same page
4. **Path separators** - Backend is Windows/.NET, but web paths use forward slashes
5. **Video IDs** - Use `@Current.Hash` or `@Model.Hash` for video identifiers, not filenames

## Useful Server-Side Model Properties

See README.md for comprehensive list. Key ones:
- `@Model.WebSiteName` - Site title
- `@Model.UserName` - Current logged-in user
- `@Model.ServerInfo` - Server status info
- `@Current.Hash` - Video file hash (used as ID)
- `@Current.AnimeTitle` / `@Current.EpisodeTitle` - Video metadata

---
> Source: [kaedei/dandanplay-libraryindex](https://github.com/kaedei/dandanplay-libraryindex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-24 -->
