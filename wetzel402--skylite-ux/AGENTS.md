
# Reuse

- Prefer reusing existing logic instead of duplicating. Before adding a new helper or composable, check: `app/composables/` (client composables e.g. useTodos, useCalendar, useSyncManager), `server/utils/` (rrule, sanitizeIntegration, icalUrl), `server/plugins/`, `server/integrations/` (google_calendar, iCal, tandoor, mealie), `app/lib/` (e.g. Prisma client).
- Prefer modifying existing files over adding new ones when the change fits there.

---
> Source: [Wetzel402/Skylite-UX](https://github.com/Wetzel402/Skylite-UX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-19 -->
