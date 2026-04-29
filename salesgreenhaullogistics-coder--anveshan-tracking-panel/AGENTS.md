
# Anveshan Tracking Panel Workspace Instructions

This workspace contains a React-based logistics tracking dashboard that processes shipment data from Google Apps Script, normalizes it through multiple mapping layers, and presents it across 17 specialized views.

## Development Setup

### Build & Test Commands
- `npm run dev` - Start Vite dev server with HMR (default port 5173)
- `npm run build` - Production build to `dist/`
- `npm run preview` - Preview production build locally
- `npm run tunnel` - Expose local server via LocalTunnel with subdomain
- `npm run deploy` - Deploy to Netlify production

### Environment Notes
- Uses embedded Node.js v22.16.0 (Windows-specific)
- No `.env` file - all configuration hardcoded
- Supports both Vercel and Netlify deployments
- CORS open to public (`Access-Control-Allow-Origin: *`)

## Architecture Overview

### Data Pipeline
Raw Google Sheets data → [api/shipmentEngine.mjs](api/shipmentEngine.mjs) (parse/normalize/filter) → [src/context/DataContext.jsx](src/context/DataContext.jsx) (React state) → Pages (17 views) → Components (charts/tables/KPIs)

### Key Components
- **Data Normalization**: Three layers in [src/utils/](src/utils/) - platform mapping, status mapping, field key mapping
- **State Management**: Tab-scoped filters in DataContext, computed unique values
- **API Actions**: shipments (filtered data), search (fuzzy multi-token), suggest (autocomplete)

### Component Boundaries
- Pages: Tab-specific data filtering and view composition
- Components: Reusable UI (Sidebar, Filters, Charts, DataTable, GlobalSearch)
- Utilities: Status predicates, date helpers, Excel export

## Key Conventions

### Data Processing
- **Platform Mapping**: Uses `consignee` field → 50+ platform names; empty = "Retail Other"
- **Status Normalization**: Exact match → case-insensitive → fuzzy pattern; preserves `rawStatus`
- **Month Derivation**: Requires valid month string + booking date (2020-2030) → "Jan'25" format
- **Tab Filtering**: Custom logic per tab (e.g., `intransit` uses `isInTransitRow()`)

### Search Architecture
- Tokenizes by whitespace/commas/semicolons
- Type detection (AWB/Invoice/PO/Ref/Mixed)
- Scoring: exact=100, prefix=80, contains=65, Levenshtein≤20%=45
- Field weighting for detected types

### Styling
- Tailwind with custom primary blue palette
- Component classes: `.kpi-card`, `.filter-bar`, `.chart-container`
- Dark sidebar theme (#0f172a-#1a1f3a)
- Compact sizing (10-11px text, 12px inputs)

## Common Pitfalls

1. **Month Derivation Returns Empty**: Always validate `row.month` before filtering
2. **Platform Lookup Uses Consignee**: Not the platform field; update [src/utils/platformMapping.js](src/utils/platformMapping.js) for new platforms
3. **RTO Status Complexity**: Three mutually exclusive states; check predicates in order
4. **Date Parsing Silent Failures**: Wrap in null checks; supports multiple formats
5. **Google Apps Script URL**: Hardcoded in 3 files - update all if changed
6. **Tab State Independence**: Filters don't persist across tabs
7. **Search Not Cached**: Expensive on large datasets
8. **Type Safety**: All fields converted to strings early

## Important Files

| File | Purpose |
|------|---------|
| [api/shipmentEngine.mjs](api/shipmentEngine.mjs) | Core query engine, data parsing, filtering |
| [src/context/DataContext.jsx](src/context/DataContext.jsx) | Global state, tab filters, unique values |
| [src/utils/statusMapping.js](src/utils/statusMapping.js) | Status normalization mappings |
| [src/utils/platformMapping.js](src/utils/platformMapping.js) | Platform name mappings |
| [src/utils/index.js](src/utils/index.js) | Helper functions, predicates, colors |
| [src/pages/Dashboard.jsx](src/pages/Dashboard.jsx) | Multi-view page pattern |
| [src/components/DataTable.jsx](src/components/DataTable.jsx) | Sortable table with export |
| [src/index.css](src/index.css) | Component styling definitions |
| [vite.config.js](vite.config.js) | Build config, proxy setup |
| [package.json](package.json) | Dependencies and scripts |

## Adding New Features

### New Status
Add to `STATUS_MAP` in [src/utils/statusMapping.js](src/utils/statusMapping.js)

### New Platform
Add to `CONSIGNEE_TO_PLATFORM` in [src/utils/platformMapping.js](src/utils/platformMapping.js)

### New Tab/Page
1. Add filter logic to `filterByTab()` in [api/shipmentEngine.mjs](api/shipmentEngine.mjs)
2. Create page component following [src/pages/InTransit.jsx](src/pages/InTransit.jsx) pattern
3. Add to routing in [src/App.jsx](src/App.jsx)

### New Search Field
Update `KEY_MAP` in [src/context/DataContext.jsx](src/context/DataContext.jsx) and parsing in [api/shipmentEngine.mjs](api/shipmentEngine.mjs)

---
> Source: [salesgreenhaullogistics-coder/anveshan-tracking-panel](https://github.com/salesgreenhaullogistics-coder/anveshan-tracking-panel) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-29 -->
