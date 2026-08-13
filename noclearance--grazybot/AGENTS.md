## Quick orientation — what this repo is

This repository is a Discord clan bot for Old School RuneScape (GrazyBot). There are two competing entrypoints: a single-file, working monolith in `bot.py` (the runtime most files reference) and an alternative modular `main.py` scaffold that expects `cogs.*` extensions. For immediate work, treat `bot.py` as the canonical runtime unless you are intentionally migrating to the `cogs` structure.

Key responsibilities:
- Event scheduling and reminders (SOTW, raffles, giveaways, PVM events, bingo)
- OSRS integration (WiseOldMan API and Hiscores)
- Persistence via PostgreSQL using `asyncpg` connection pool
- AI-generated embed text via Google Gemini (`google.generativeai`) with JSON embed parsing + explicit fallbacks

## Where to start reading
- `bot.py` — main runtime: commands, background tasks (`event_manager`, `periodic_event_reminder`), DB schema creation, persistent view re-registration, web health endpoint and start logic.
- `config.py/config.py` — environment-driven config helpers used elsewhere.
- `requirements.txt/requirements.txt` — third-party packages the project expects (aiohttp, asyncpg, google-generative-ai, python-dotenv, PIL/Pillow implied by imports).
- `scaffolds/cog_template.py` — example for creating a new cog if you elect to move logic into extensions.
- `cogs.py/*` — collection of small helper modules (dbpool, helpers_ai, helpers_utils, etc.) — read for utility patterns.

## Run / dev setup (discoverable from repo files)
Environment variables (required by `bot.py`):
- `TOKEN`, `DATABASE_URL`, `DEBUG_GUILD_ID`, `SOTW_ROLE_ID`, `SOTW_CHANNEL_ID`, `RAFFLE_CHANNEL_ID`, `RECAP_CHANNEL_ID`, `ANNOUNCEMENTS_CHANNEL_ID`, `WOM_CLAN_ID`, `WOM_VERIFICATION_CODE`, `GEMINI_API_KEY` (and optional `PORT`, `TASKS_FILE`).

Minimal run example (PowerShell):
```powershell
# load env vars (for example using a .env) then:
python bot.py
```

Notes:
- `bot.py` will create all required DB tables on startup (see `setup_database_pool()`), so the only DB precondition is a reachable Postgres instance referenced by `DATABASE_URL`.
- The bot runs a small aiohttp web server for health checks on `PORT` (default 10000).

## Code patterns to follow (concrete, repo-specific)
- Database access: always use `async with bot.db_pool.acquire() as conn:` then `await conn.fetchrow`/`fetch`/`execute`. Look at `award_points`, `raffle` and `pvm` command implementations for examples.
- Background tasks: long-running logic sits inside `@tasks.loop(...)` functions; they assume UTC datetimes. Use `bot.wait_until_ready()` at top of loops.
- Persistent views: `GiveawayView` / `PvmEventView` are re-registered in `on_ready()` by reading saved message IDs from the DB. To add a new persistent interactive view, persist the message ID and re-register it in `on_ready()` the same way.
- External API calls: prefer creating a new `aiohttp.ClientSession()` per caller as shown; use `asyncio.gather` to parallelize multiple requests (see `fetch_all_stats_async`).
- Blocking ops (PIL image generation, file open): dispatch to threadpool via `await asyncio.to_thread(...)` (see `_generate_bingo_image_sync` + `update_bingo_board_post`).

## AI / Gemini usage
- Calls use `google.generativeai` with `ai_model.generate_content_async(...)`. Responses are parsed as JSON for embed fields (see `generate_announcement_json`).
- There is explicit fallback text in `EMBED_FALLBACKS` used when parsing fails — ensure new prompts preserve this fallback pattern.

## Common change tasks and concrete examples
- Add a new slash command: copy the pattern used near `/raffle start` or `/bingo start` — commands are registered with `@bot.command` or `bot.create_group(...); @group.command(...)`.
- Add DB columns/table: update `setup_database_pool()` inside `bot.py` (it runs CREATE TABLE IF NOT EXISTS on startup). Tests are not present; ensure migrations are backwards-compatible.
- Add a new event type: follow the `pvm_events` pattern: DB table, commands to create the event, a persistent view for signups, and event manager alarms in `event_manager`.

## Gotchas & repo-specific conventions
- Timezone: the code uses UTC everywhere (datetime with tzinfo=timezone.utc and Discord timestamps). Always store and compare with UTC.
- Two entrypoints exist — editing `main.py` without aligning `cogs.*` layout will not affect `bot.py`. Decide which runtime is canonical before refactors.
- Many functions assume certain environment variables exist and will raise if missing (`bot.py` validates them near top). When testing locally, set required vars or edit `config` helpers.
- Item mapping loads from external API at startup with retry/backoff (`load_item_mapping`). Avoid blocking startup on this in tests (mock it).
- Message IDs and channel IDs are saved into the DB and used later for editing messages; be careful when deleting or moving those messages in Discord.

## Files to check when making changes
- `bot.py` — primary behavior
- `config.py/config.py` — env loader and validator
- `requirements.txt/requirements.txt` — ensure dependencies defined
- `scaffolds/cog_template.py` — pattern for new cogs
- `cogs.py/helpers_ai.py`, `cogs.py/dbpool.py`, `supabas.py/supabase.py` — utility helpers and integration code

If anything in this brief guide is unclear or you want it expanded (examples for writing a new cog, tests, or a migration plan to the `cogs/` layout), tell me which section to expand and I will iterate.

---
> Source: [noclearance/Grazybot](https://github.com/noclearance/Grazybot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-13 -->
