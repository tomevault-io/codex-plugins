# Copilot Instructions for Tampa Meetings

This document provides context and guidelines for AI assistants working on this codebase.

---

## Project Overview

**Tampa Meetings** is a civic transparency tool that scrapes, processes, and publishes Tampa City Council meeting records. It combines:

1. **Agenda Scraper** (Node.js) — Extracts structured data from Hyland OnBase meeting system
2. **Transcript Processor** (Python) — Converts ALL CAPS transcripts to sentence case with NER
3. **Static Site** (Eleventy) — Generates accessible HTML pages
4. **API** (Datasette) — Serves meeting data as JSON

---

## Key Technical Decisions

### Architecture

| Component   | Technology       | Why                                |
| ----------- | ---------------- | ---------------------------------- |
| Static Site | Eleventy         | Simple, fast, template flexibility |
| API         | Datasette        | Zero-code JSON API from SQLite     |
| Hosting     | Cloudflare Pages | Free, global CDN                   |
| Documents   | Cloudflare R2    | S3-compatible, free egress         |
| Database    | SQLite           | Single file, portable, FTS5 search |

### Code Patterns

- **Accessibility first** — WCAG 2.1 AA is the primary constraint
- **Semantic HTML** — No divitis; use proper elements
- **Native web components** — Prefer vanilla JS over frameworks
- **Progressive enhancement** — Core content works without JS
- **Small functions** — Keep functions focused and testable

---

## Directory Structure

```
tampa-meetings/
├── pipeline/           # Data processing (Node.js + Python)
│   ├── scrapers/       # Web scraping
│   ├── processors/     # NER, case conversion
│   └── scripts/        # Utilities
├── site/               # Eleventy static site
│   ├── src/            # Templates
│   └── public/         # Static assets
├── data/               # Scraped data
│   ├── agendas/        # JSON files
│   └── meetings.db     # SQLite
└── docs/               # Documentation
```

---

## Code Style Guidelines

### JavaScript (Node.js scrapers)

```javascript
// ✅ Good: Small, focused functions
async function fetchMeetingPage(meetingId) {
  const url = buildMeetingUrl(meetingId);
  const response = await fetch(url);
  return response.text();
}

// ✅ Good: Descriptive names
const fileNumberPattern = /[A-Z]{2,4}\d{2}-\d{4}/g;

// ❌ Bad: Magic numbers without context
const delay = 3000;

// ✅ Good: Named constants
const RATE_LIMIT_DELAY_MS = 3000;
```

### Python (processors)

```python
# ✅ Good: Type hints
def process_transcript(text: str, entities: list[Entity]) -> ProcessedTranscript:
    pass

# ✅ Good: Docstrings for complex functions
def extract_chapters(video_id: str) -> list[Chapter]:
    """
    Extract chapter markers from YouTube video description.

    Returns list of Chapter objects with title and timestamp.
    Raises YouTubeAPIError if video not found.
    """
    pass

# ❌ Bad: Bare except
try:
    result = risky_operation()
except:
    pass

# ✅ Good: Specific exceptions
try:
    result = risky_operation()
except RequestException as e:
    logger.error(f"Request failed: {e}")
    raise
```

### HTML/Nunjucks (Eleventy templates)

```html
<!-- ✅ Good: Semantic structure -->
<article class="meeting">
  <header>
    <h1>{{ meeting.title }}</h1>
    <time datetime="{{ meeting.date }}">{{ meeting.date | formatDate }}</time>
  </header>
  <main>
    <section class="agenda">
      <h2>Agenda Items</h2>
      <!-- items -->
    </section>
  </main>
</article>

<!-- ❌ Bad: Div soup -->
<div class="meeting">
  <div class="header">
    <div class="title">{{ meeting.title }}</div>
  </div>
</div>
```

### CSS

```css
/* ✅ Good: Logical properties for RTL support */
.sidebar {
  margin-inline-start: 1rem;
  padding-block: 0.5rem;
}

/* ✅ Good: Custom properties for theming */
:root {
  --color-primary: #1a5f7a;
  --spacing-md: 1rem;
}

/* ❌ Bad: Fixed pixel widths for text containers */
.content {
  width: 800px;
}

/* ✅ Good: Fluid, responsive widths */
.content {
  max-width: 65ch;
  width: 100%;
}
```

---

## Domain Knowledge

### Meeting Types

| Code | Full Name                      | Frequency         |
| ---- | ------------------------------ | ----------------- |
| CC   | City Council                   | Weekly (Thursday) |
| CRA  | Community Redevelopment Agency | Thursdays         |
| EVE  | Evening Session                | Thursday Evening  |
| WS   | Workshop                       | Thursdays         |
| SP   | Special Meeting                | As needed         |

### File Number Format

City uses format like `CRA24-2242`, `CC25-0015`:

- First 2-4 letters: Meeting type
- 2 digits: Year
- 4 digits: Sequential number

```javascript
// Pattern for matching
const FILE_NUMBER_PATTERN = /([A-Z]{2,4})(\d{2})-(\d{4})/;
```

### Document Types

- **Staff Report** — City staff analysis and recommendation
- **Ordinance** — Proposed law change
- **Resolution** — Policy statement
- **Agreement** — Contract or MOU
- **Map/Exhibit** — Supporting visuals (often scanned PDFs)

---

## Common Tasks

### Adding a New Meeting Type

1. Update `MEETING_TYPES` constant in `pipeline/scrapers/config.js`
2. Add parsing rules if format differs
3. Update Eleventy filters for display

### Debugging Scrape Failures

1. Check if city site HTML structure changed
2. Look for rate limiting (add delays)
3. Check for JavaScript-rendered content (may need Playwright)

### Updating Datasette Schema

1. Modify table creation in `pipeline/scripts/build-database.py`
2. Rebuild database: `python pipeline/scripts/build-database.py`
3. Redeploy: `./pipeline/scripts/deploy-datasette.sh`

---

## Testing

### Manual Testing Checklist

- [ ] Keyboard navigation works for all interactive elements
- [ ] Screen reader announces content logically
- [ ] Color contrast passes WCAG AA (4.5:1 for text)
- [ ] Site works with JavaScript disabled (core content)
- [ ] PDFs open correctly from R2 links
- [ ] Mobile layout is usable

### Automated Checks

```bash
# Lint JavaScript
npm run lint

# Build site (catches template errors)
cd site && npm run build

# Check for accessibility issues
npx pa11y-ci ./site/_site/**/*.html
```

---

## Python Environment

All Python code runs from a single virtualenv at `transcript-cleaner/processor/venv/`.

**Always activate before running any Python command:**

```bash
source pipeline/activate.sh
```

This applies to transcript processing, video pipeline, entity rebuilds, and any `pip install`. Scripts like `process-meeting.sh` and `discover.py` auto-activate, but running Python scripts directly (e.g., `python3 src/youtube_fetcher.py` or `python3 scripts/build/process_video.py`) requires manual activation first.

---

## External Services

| Service          | Purpose            | Credentials Location                              |
| ---------------- | ------------------ | ------------------------------------------------- |
| Cloudflare Pages | Static hosting     | Managed in CF dashboard                           |
| Cloudflare R2    | Document storage   | `S3_*` env vars, `docs.meetings.tampamonitor.com` |
| Vultr VPS        | Datasette hosting  | SSH key                                           |
| YouTube Data API | Chapter extraction | `YOUTUBE_API_KEY` env var                         |

---

## Video Pipeline

The transcript processor includes a video pipeline that matches YouTube recordings to meetings and calculates time offsets for clickable timestamps.

### Key modules

| Module                                         | Purpose                                                                          |
| ---------------------------------------------- | -------------------------------------------------------------------------------- |
| `src/meeting_type_detector.py`                 | Auto-detects CRA / Workshop / Evening / City Council from transcript             |
| `src/youtube_fetcher.py`                       | Searches YouTube Data API, saves `video_mapping_<ID>.json`                       |
| `scripts/build/match_whisper_to_transcript.py` | Whisper-based offset calculation, auto-saves to mapping                          |
| `scripts/build/transcribe_with_whisper.py`     | Standalone Whisper transcription                                                 |
| `src/transcript_gap_detector.py`               | Detects multi-part boundaries from timestamp gaps, saves `transcript_start_time` |
| `src/html_generator.py`                        | Generates HTML pages with video-synced timestamps                                |

### Design constraints

- **No `youtube-transcript-api`** — causes IP bans. Use Whisper for offset detection instead.
- **YouTube Data API quota** — ~102 units per meeting (~10,000/day free). Keep searches minimal.
- **yt-dlp rate limiting** — add 5-10s delay between consecutive audio downloads.
- **Speaker names from official transcripts only** — no ML transcript service provides speaker attribution.
- **Multi-part videos** — meetings can have 2-3 videos (morning/afternoon/evening, or streaming interruptions). Each video gets an independent `offset_seconds`.
- **Long intros** — Part 2+ videos may have 10+ minutes of countdown/music before speech. Whisper sample duration must account for this.

### Multi-part video mapping schema

```json
{
  "meeting_id": 2645,
  "meeting_date": "2025-11-13",
  "meeting_type": "CRA",
  "videos": [
    {
      "video_id": "SocxtU6vTKc",
      "part": 1,
      "offset_seconds": 552,
      "transcript_start_time": null,
      "chapters": [...]
    }
  ]
}
```

### Pipeline plan

See `transcript-cleaner/processor/docs/VIDEO_PIPELINE.md` for the full phased plan (all 5 steps complete).

---

## When Making Changes

1. **Check the implementation plan** — See `docs/IMPLEMENTATION_PLAN.md` for architectural decisions
2. **Prioritize accessibility** — This is non-negotiable
3. **Test on real data** — Use actual meeting JSON, not mocks
4. **Keep dependencies minimal** — Prefer standard library over npm packages
5. **Document breaking changes** — Update this file and README

---

## Questions to Ask

Before making significant changes, consider:

1. Does this affect accessibility?
2. Does this change the data schema?
3. Will this break existing meeting pages?
4. Is there a simpler approach?
5. Does this align with the implementation plan?

---

_Last updated: March 8, 2026_

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/miklb)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/miklb)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
