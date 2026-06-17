# SyncSong

## Overview

SyncSong is a web app for creating and editing synced lyrics in LRC format. The LRC
file format is commonly used for displaying synced lyrics with music, where each
line of lyrics is prefixed with a timestamp controlling when the line is displayed.

The app runs entirely in the user's browser - files are never uploaded anywhere.
Users can:

1. **Add a song** - Drag/drop or browse for audio files (MP3, FLAC, OGG, M4A, WAV)
2. **Add lyrics** - Search LRCLIB, import files, or type/paste lyrics directly
3. **Sync lyrics** - Adjust timestamps using keyboard controls while listening
4. **Export** - Download LRC file, copy to clipboard, or publish to lrclib.net

## Live Site

The app is deployed to GitHub Pages at: https://syncsong.net/

## Tech Stack

- **Vite** - Build tool and dev server with PWA plugin
- **Tailwind CSS** - Utility-first CSS framework with dark theme
- **Vanilla JavaScript (ES Modules)** - No framework, keeping it lightweight
- **WaveSurfer.js** - Audio waveform visualization
- **music-metadata** - Extract metadata from audio files
- **Transformers.js** - Client-side AI for optional lyrics transcription (lazy-loaded)
- **Web Audio API / HTML5 Audio** - Native browser audio playback
- **File API / Drag & Drop API** - Native browser file handling

## Project Structure

```
SyncSong/
├── index.html              # Main HTML with Tailwind classes and Open Graph meta tags
├── vite.config.js          # Vite configuration with PWA manifest
├── tailwind.config.js      # Tailwind configuration with custom primary colors
├── postcss.config.js       # PostCSS configuration
├── package.json            # Dependencies
├── src/
│   ├── main.js             # Entry point, initializes app
│   ├── style.css           # Tailwind imports + custom component classes
│   ├── lrc-parser.js       # Parse/serialize LRC format, timestamp utilities
│   ├── audio-player.js     # AudioPlayer class - playback controls, waveform, events
│   ├── editor.js           # LyricEditor class - timestamp adjustment, line selection
│   ├── transcriber.js      # Lazy-loaded Whisper-based speech-to-text (optional)
│   └── ui.js               # UI class - DOM manipulation, event handlers, drag/drop
├── public/
│   ├── favicon.ico         # Site favicon
│   ├── splitwave.svg       # App logo
│   ├── social-card.png     # Open Graph social sharing image
│   ├── social-card.svg     # Source for social card
│   └── pwa/                # PWA icons (various sizes)
├── test/
│   └── data/               # Test audio and LRC files for Playwright testing
│       ├── The Dream.flac
│       └── The Dream.lrc
└── .github/
    ├── copilot-instructions.md
    ├── javascript.instructions.md
    └── workflows/
        └── deploy.yml      # GitHub Actions workflow for GitHub Pages deployment
```

## Architecture

The site is served as a static website with a PWA manifest so users can install
it on their devices.

### Core Classes

- **AudioPlayer** (`audio-player.js`): Wraps WaveSurfer.js for waveform visualization
  and audio playback. Provides play/pause/seek and emits callbacks for timeupdate,
  play, pause, ended, durationchange.

- **LyricEditor** (`editor.js`): Manages lyrics data and metadata, handles line selection,
  timestamp adjustments (±0.1s increments), tracks currently playing line index.

- **UI** (`ui.js`): Connects AudioPlayer and LyricEditor to the DOM, handles all event
  listeners (drag/drop, keyboard, mouse wheel, clicks), renders the 4-step wizard UI.

### Data Flow

1. User drops/pastes lyrics → `lrc-parser.js` parses to `{ metadata, lines }` → `LyricEditor.loadLyrics()`
2. User drops audio → `AudioPlayer.loadFile()` → triggers duration callback → auto-sync if unsynced
3. Playback → `onTimeUpdate` → `editor.updatePlayingLine()` → auto-select current line
4. User adjusts → `editor.adjustTimestamp()` → `player.seek()` → UI updates timestamp display
5. Export → `editor.toLRC()` → copy to clipboard or download as file

### User Workflow (4-Step Wizard)

1. **Step 1: Song** - Drop or browse for audio file, metadata extracted automatically
   - Optional: AI transcription with Whisper to auto-generate lyrics with timestamps
2. **Step 2: Lyrics** - Search LRCLIB, open file, or paste/type lyrics; edit metadata
3. **Step 3: Sync** - Click lines to jump, use ↑/↓ arrows to navigate, ←/→ to adjust timestamps
4. **Step 4: Export** - Preview, copy, download, or publish to lrclib.net

## Coding Conventions

- Use ES modules (`import`/`export`)
- Classes for stateful components (AudioPlayer, LyricEditor, UI)
- Pure functions for utilities (lrc-parser.js)
- Callbacks instead of EventEmitter for simplicity
- Tailwind utility classes in HTML, custom components in `@layer components`
- Dark theme by default (slate color palette)
- Use `primary-*` color scale for accent colors (indigo-based)

## LRC Format Reference

```
[ar:Artist Name]
[ti:Song Title]
[al:Album Name]
[by:LRC Author]

[00:12.34]First line of lyrics
[00:15.67]Second line of lyrics
[01:02.00]Timestamp format is [mm:ss.xx] (hundredths of seconds)
```

Supported metadata tags: `ar` (artist), `ti` (title), `al` (album), `by` (author), `offset`

See [LRC File Format](https://en.wikipedia.org/wiki/LRC_(file_format)) for the full specification.

## External Services

- **LRCLIB** (https://lrclib.net/) - Community lyrics database
  - Search for existing synced/unsynced lyrics
  - Publish synced lyrics to share with the community
  - API docs: https://lrclib.net/docs

## Deployment

The app is deployed to GitHub Pages via GitHub Actions. The workflow in
`.github/workflows/deploy.yml` builds the Vite project and deploys on pushes
to the `main` branch.

## Development

```bash
npm install      # Install dependencies
npm run dev      # Start dev server
npm run build    # Build for production
npm run preview  # Preview production build
```

## Future Enhancements

- Multiple timestamp selection for batch adjustments
- Undo/redo functionality
- Export to other subtitle formats (SRT, VTT)

## Test Files

Test audio and LRC files are available under `test/data/` for testing the frontend
through the Playwright MCP.

---
> Source: [joshooaj/SyncSong](https://github.com/joshooaj/SyncSong) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-17 -->
