# AnimeStream Addon - AI Coding Instructions

**IMPORTANT**: Do NOT commit or push changes to `.github/copilot-instructions.md` to the repository. This file is for local AI guidance only.

## ⚠️ CRITICAL: Development Server Rules

**DO NOT START THE WRANGLER SERVER YOURSELF!**
- The server is ALWAYS running in the background terminal
- Logs are output to `cloudflare-worker/logs1.txt`
- If you need to test changes, ASK THE USER to restart the server
- Only read `logs1.txt` to check server output
- Starting the server wastes tokens and interrupts the user's workflow

## Architecture Overview

AnimeStream is a **Stremio addon** for anime content that uses a **pre-bundled database** of anime from Kitsu/IMDB with **integrated streaming via AllAnime**. It provides custom catalogs with instant loading and direct video streams.

### Core Layers
- **Cloudflare Worker** (`cloudflare-worker/worker-github.js`): Main deployed worker with catalog + streaming
- **AllAnime Integration**: Direct GraphQL API calls for stream discovery
- **Database** (`data/`): Pre-bundled catalog on GitHub for instant loading
- **Scrapers** (`scrapers/`): Standalone AllAnime scraper (deprecated, now integrated)

### Critical Design Decisions
1. **NOT using stremio-addon-sdk's serveHTTP** - bypasses 8KB manifest limit
2. **Database-first architecture** - pre-bundled `catalog.json` served from GitHub
3. **Direct AllAnime API** - XOR-encrypted URLs (key=56), GraphQL API
4. **No Worker-to-Worker calls** - AllAnime integration is inline to avoid Cloudflare error 1042

## Data Flow

```
Catalog Request → GitHub catalog.json → catalog handler → formatAnimeMeta → Stremio

Stream Request → findAnimeByImdbId → searchAllAnime → fuzzy match → getEpisodeSources → Stremio
```

## Key Patterns

### Anime IDs
IDs use IMDB format: `tt{imdb_id}` (e.g., `tt13706018` for Spy x Family)
- Stremio provides IDs as `tt13706018:1:5` (imdb:season:episode)

### AllAnime XOR Decryption
AllAnime encrypts source URLs with XOR key 56:
```js
function decryptSourceUrl(input) {
  const str = input.startsWith('--') ? input.slice(2) : input;
  let result = '';
  for (let i = 0; i < str.length; i += 2) {
    const num = parseInt(str.substr(i, 2), 16);
    result += String.fromCharCode(num ^ 56);
  }
  return result;
}
```

### Fuzzy Title Matching
Uses Levenshtein distance to match catalog titles to AllAnime search results:
- Exact match: 100 points
- Substring match: 80 points  
- Fuzzy similarity: 0-90 points
- TV bonus: +3, Movie bonus: +2
- Minimum score to accept: 60

## Catalog Types

1. **Top Rated** - Sorted by rating
2. **Season Releases** - Current season anime
3. **Currently Airing** - Anime currently broadcasting
4. **Movies** - Anime movies

## AllAnime GraphQL API

### Search Anime
```graphql
query ($search: SearchInput!, $limit: Int) {
  shows(search: $search, limit: $limit, translationType: "sub", countryOrigin: "JP") {
    edges { _id name englishName nativeName type score status }
  }
}
```

### Get Episode Sources
```graphql
query ($showId: String!, $translationType: VaildTranslationTypeEnumType!, $episodeString: String!) {
  episode(showId: $showId, translationType: $translationType, episodeString: $episodeString) {
    episodeString sourceUrls
  }
}
```

## Deployment

### Cloudflare Worker
```powershell
cd cloudflare-worker
npx wrangler deploy  # Deploys worker-github.js
```

### Debug Endpoint
`/debug/stream/{imdb_id}:{season}:{episode}` - Returns detailed matching info for troubleshooting

## Environment Variables
- `ENVIRONMENT` - Set to `production` on Cloudflare

## Stream Sources
AllAnime provides direct video URLs from:
- **fast4speed.rsvp** - Primary CDN (needs Referer header)
- Other direct sources as fallback

Streams include both **SUB** (Japanese with subtitles) and **DUB** (English dubbed) when available.

---
> Source: [Zen0-99/animestream-addon](https://github.com/Zen0-99/animestream-addon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-02 -->
