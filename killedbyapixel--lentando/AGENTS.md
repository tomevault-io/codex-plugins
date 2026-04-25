# Lentando - AI Agent Instructions

## Project Overview
Zero-friction substance use & habit tracker. PWA, vanilla JS (no frameworks), mobile-first, offline-capable. localStorage is primary storage; Firebase is optional sync/backup. Never block UI on network.

## Architecture
- `code.js` — All app logic, rendering, badge calculations, event tracking
- `index.html` — Single-page app with inline CSS
- `firebase-sync.js` — Firebase Auth + Firestore merge logic (ES module)
- `sw.js` — Service worker (cache: `lentando-v{N}`, increment on deploy)
- `zzfx.js` — Sound effect system, do not edit.
- `build.js` — Minifies JS via terser, copies static files to `dist/`
- `local/` — Dev notes, not deployed

## Storage & Data
**localStorage keys:** `ht_events`, `ht_settings`, `ht_todos`, `ht_theme`, `ht_badges`, `ht_login_skipped`, `ht_data_version`, `ht_deleted_ids`, `ht_deleted_todo_ids`, `ht_cleared_at`, `ht_last_updated`

**DB module** wraps localStorage with in-memory caches (`DB._events`, `DB._settings`, `DB._dateIndex`). The `_dateIndex` is a lazy `Map<dateKey, events[]>`.

**Data model:**
- **Events** — `{id, type, ts, ...}` where type is `used`, `resisted`, or `habit`
- **Badges** — `{todayDate, todayBadges, yesterdayBadges, lifetimeBadges, todayUndoCount, appStartTs, earliestEventTs}`. `appStartTs` is write-once (first app open). `earliestEventTs` is recalculated from events (null if none). Badge anchor = `min(earliestEventTs, appStartTs)`. Merge strategy: max count per badge ID.

### Cache Invalidation (Critical)
`firebase-sync.js` uses `invalidateDBCaches()` after cloud merges — this nulls `DB._events`, `DB._settings`, `DB._dateIndex`. Must happen **inside** the function that writes to localStorage, not after it returns.

## Key Patterns

### Event Lifecycle
`logUsed()` / `logResisted()` / `logHabit()` → `DB.addEvent()` → `calculateAndUpdateBadges()` → `render()`. Cloud sync fires automatically: `DB.saveEvents()` calls `FirebaseSync.onDataChanged()` internally (debounced 3s push).

### Badge System
`calculateAndUpdateBadges()` recalculates all badges on every event change. Define new badges in `BADGE_DEFINITIONS`, add logic using `addBadge(condition, 'badge-id')` inside `Badges.calculate()`.

### Time Boundaries
- **Day boundary:** Calendar days (midnight), BUT gap calculations exclude gaps crossing 6am (`EARLY_HOUR = 6`)
- **Gap calculations:** All gap metrics (longest gap, average gap, hour gaps) exclude gaps crossing 6am to filter out overnight sleep
- **Skip badges:** Eligible once past start time

### Event Consolidation
Events older than 60 days (`CONSOLIDATION_DAYS`) are automatically merged to save localStorage space. Runs once on app start via `consolidateOldEvents()`. A shared `consolidationGroupKey(e)` function produces the grouping key for both initial consolidation and past-event absorption. Group rules:
- **Used events**: grouped by substance + reason — amounts summed, method becomes `'mixed'` if varied, reason preserved per group
- **Resisted events**: grouped by trigger — intensity summed, trigger preserved per group
- **Habit events**: grouped by habit type — water uses `count` field (no minutes), other habits sum minutes (5-min default for untimed)

Merged events get `consolidated: N` (integer count of how many original events were merged) + `modifiedAt` timestamp. Single consolidated events have `consolidated: 1`; multi-event merges have the total count. Absorbing a new past event into a keeper increments the count. Any truthy `consolidated` value means the event has been consolidated. The most recent event in each group becomes the keeper. If strays reappear from sync (keeper already consolidated), they're silently discarded. Add-past-event to consolidated days: absorbed directly into keeper at call site (`saveCreateModal`) instead of creating a separate event — increments count and sets `modifiedAt` so the absorb survives a sync pull before the push fires.

### Deletion Model
Soft-delete via tombstones (`ht_deleted_ids` / `ht_deleted_todo_ids`): maps of `{id: deletedAtTimestamp}`, cleaned after 90 days. Bulk clear uses `ht_cleared_at` timestamp — events with uid created before that timestamp are discarded.

### Firebase Sync
- **Pull:** `onAuthStateChanged` → `pullFromCloud()` → merge tombstones (union) → merge events (union by ID, per-event `modifiedAt` wins conflicts) → merge settings (recency-based) → max badges → `invalidateDBCaches()` → `continueToApp()`
- **Push:** Data change → `onDataChanged()` → debounced 3s → `pushToCloud()`
- **Focus-pull:** App regains focus → flush pending push, then pull fresh data

## Profiles
`ADDICTION_PROFILES`: cannabis, alcohol, smoking, custom. Each has substances, methods, amounts, icons. Selected in onboarding, access via `getProfile()`.

## Development

### Commands
```bash
npm run build      # Minifies to dist/
npm run clean      # Removes dist/
npm run lint       # ESLint across code.js, firebase-sync.js, sw.js, build.js
```

### Test Data Generators
Defined at bottom of `code.js`, gated behind `debugMode`. When `debugMode = true`, they are auto-exposed on `window`:
- `generateAllTestData()` — Mix of everything
- `generateUseEvent(7)` — Single use event 7 days ago

## Common Gotchas
- **Don't pre-load DB in DOMContentLoaded** — Firebase auth flow handles it
- **Always `escapeHTML()`** before innerHTML with user text
- **Old events may lack new fields** — Use `evt?.field ?? default`
- **6am is a boundary for gaps** — Critical to prevent overnight sleep counting as a long gap. This way we only track gaps during the day.
- **Vanilla JS only** — ES6+ OK, no frameworks, no JSX, no build transforms
- **Mobile-first CSS** — Test at 320px viewport
- **Clear intervals before re-setting** — Prevents timer leaks

---
> Source: [KilledByAPixel/Lentando](https://github.com/KilledByAPixel/Lentando) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-25 -->
