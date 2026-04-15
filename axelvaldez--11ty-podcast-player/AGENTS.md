# 11ty Podcast Website Project

## Project Overview
This is an 11ty (Eleventy) static site generator project that fetches a podcast RSS feed and generates a modern web podcast player with show information, episode pages, and an interactive audio player.

## Project Structure
- `_data/` - Contains data files that fetch and parse the podcast RSS feed, plus translation files
- `_includes/` - Contains layouts and partial templates
- `src/` - Source files for pages and content
- `css/` - Stylesheet files
- `js/` - Client-side JavaScript for player functionality
- `.eleventy.js` - 11ty configuration file

## Setup Instructions
1. Install dependencies: `npm install`
2. Configure podcast RSS feed URL and settings in `_data/podcast.js`
3. Build the site: `npm run build`
4. Serve locally: `npm start`

## Key Features
- Modern three-column web player layout
- Persistent audio playback with skip controls
- Multi-language support (EN, ES, FR, IT, PT, DE)
- Episode pagination and details sidebar
- Subscription platform links

## Technologies Used
- Eleventy (11ty) - Static site generator
- Node.js - JavaScript runtime
- xml2js - RSS feed parsing
- node-fetch - Fetching RSS feed data
- Nunjucks - Templating engine
- HTML5 Audio API - Audio playback

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/axelvaldez) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
