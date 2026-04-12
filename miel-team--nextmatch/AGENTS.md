
## Database Authority Rules

- Database is the single source of truth.
- Do not store persistent filters in localStorage.
- Search filters must be read from user_search_preferences.
- SmartMatches must always load preferences from DB.
- Never maintain multiple sources of filter state.
- All preference changes must invalidate smart_match_cache.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MIEL-TEAM)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/MIEL-TEAM)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
