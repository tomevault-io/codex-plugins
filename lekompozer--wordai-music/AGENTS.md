# GitHub Copilot Instructions — WordAI Music

## Project Overview
`wordai-music` is a **standalone Tauri v2 desktop app** (macOS/Windows/Linux) for music playback.
Built with Next.js 15 (static export) + React 19 + TypeScript + Tailwind CSS.

**Future**: `MusicPlayerMobile.tsx` will become a Capacitor iOS/Android app.

## Tech Stack
- **Desktop**: Tauri v2, Rust
- **Frontend**: Next.js 15 (static export `output: 'export'`), React 19, TypeScript
- **Styling**: Tailwind CSS (dark mode first)
- **Auth**: Firebase + Google OAuth (Tauri system browser via `google_auth.rs`)
- **Backend API**: `https://ai.wordai.pro` (same as main wordai.pro)

## Key Differences from wordai (main repo)

| | wordai (main) | wordai-music |
|---|---|---|
| Shell | HomeShell (full nav) | MusicHeader (compact title bar only) |
| Routing | Many pages | Single page (`/`) |
| Frontend delivery | Vercel (SSR/ISR) | Static export (`out/`) loaded by Tauri |
| Tauri identifier | `pro.wordai.desktop` | `pro.wordai.music` |
| Dev port | 3000 | 3001 |
| Window style | Decorated | Undecorated (custom header) |
| Update endpoint | `…/desktop/latest.json` | `…/desktop-music/latest.json` |

## Critical Rules (same as wordai)

### Tauri API — ALWAYS dynamic import
```ts
// ✅ CORRECT
if (isTauriDesktop()) {
    const { invoke } = await import('@tauri-apps/api/core');
    await invoke('some_command');
}
// ❌ NEVER — breaks Next.js static export
import { invoke } from '@tauri-apps/api/core';
```

### Static Export Constraints
- **No `<Image>` with remote URLs** — use `<img>` directly (static export can't optimize remote images)
- **No `useRouter` push to dynamic routes** — single page app
- **No server components that fetch data** — all data fetching is client-side
- **No API routes** (`src/app/api/`) — not supported in static export

### Modal Pattern (same as wordai)
- Backdrop: `fixed inset-0 ... flex items-center justify-center` — NO `overflow-y-auto`
- Card: `max-h-[90vh] flex flex-col`
- Body: `flex-1 overflow-y-auto` — only the body scrolls
- Always use `createPortal(content, document.body)`

### Window Dragging
- `MusicHeader` has `data-tauri-drag-region` — only that element is draggable
- Buttons inside header need `style={{ WebkitAppRegion: 'no-drag' }}` to be clickable

## File Structure
```
src/
  app/
    layout.tsx              ← Root layout (AppProviders)
    page.tsx                ← Loads MusicApp dynamically
    globals.css             ← Tailwind + html/body overflow:hidden
  components/
    MusicApp.tsx            ← Shell: MusicHeader + player router
    MusicHeader.tsx         ← Title bar + Login + Language + Theme
    MusicPlayerClient.tsx   ← Desktop player (AudioMotion, fullscreen YouTube)
    MusicPlayerMobile.tsx   ← Mobile player (scroll-snap, iOS audio session)
    MusicSidebar.tsx        ← Channel browser, playlist, YouTube import
    HomeShell.ts            ← Stub for HomeSidebarCollapsedCtx (always collapsed=true)
  contexts/
    AppContext.tsx           ← ThemeProvider + LanguageProvider (stripped)
    AppProviders.tsx         ← Combines all providers
    WordaiAuthContext.tsx    ← Firebase auth (Tauri + Web + Capacitor)
  services/
    musicService.ts
    musicPlaylistService.ts
    musicChannelService.ts
  lib/
    wordai-firebase.ts      ← Firebase init (shared project with wordai.pro)
    audioCache.ts

src-tauri/
  src/
    lib.rs                  ← run(), window setup, invoke_handler
    google_auth.rs          ← open_google_auth command
    main.rs                 ← entry point, Linux EGL fix
  tauri.conf.json           ← identifier: pro.wordai.music, frontendDist: ../out
  Cargo.toml
  build.rs                  ← auto-increment build_number.txt, bake OAuth creds
  capabilities/main.json

.github/workflows/
  build-desktop.yml         ← Cross-platform build on tag push (macOS/Windows/Linux)
  release.yml               ← Manual: bump version, tag, trigger build

scripts/
  dev.sh                    ← npm run dev:desktop
  tauri-dev.sh              ← beforeDevCommand (start Next.js on 3001)
  build-desktop.sh          ← production build
```

## Development

```bash
# Install
npm install

# Dev (Tauri hot reload)
npm run dev:desktop  # or bash scripts/dev.sh

# Production build
bash scripts/build-desktop.sh
```

`.env.local` required — see `.env.local.example`.

## Release Workflow
1. GitHub Actions → **Release WordAI Music** → select bump type
2. Automatically bumps version in `tauri.conf.json` + `Cargo.toml`, creates tag
3. `build-desktop.yml` triggers on tag → builds all 4 platforms in parallel
4. GitHub Release created as draft → review + publish

## Color / UI Conventions
Same dark palette as wordai: `bg-gray-800`, `bg-gray-900`, `bg-[#06060f]`, `text-white`.
Music player uses custom gradients per track (defined in `TRACK_VISUAL_THEMES` in `MusicPlayerClient.tsx`).

## Audio Patterns

### Video Pooling (YouTube fullscreen continuity)
Single global `<iframe>` never unmounted. Track changes via `postMessage loadVideoById`.
See `MusicPlayerClient.tsx` — `desktopYtIframeRef` + `desktopGlobalYtId`.

### iOS Lock-Screen Audio
Keep `<audio>` alive with silent MP3 loop when YouTube/TikTok is active.
See `MusicPlayerMobile.tsx` — `audio.src = 'data:audio/mpeg;base64,...'` pattern.

### Crossfade
Pure volume crossfade via `rampVolume()` helper — see `MusicPlayerClient.tsx`.

## Commit & Push
```bash
git add -A && git commit -m "feat: ..." && git push
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lekompozer) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-11 -->
