# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Running the server

```bash
cd ~/qa-search
.venv/bin/uvicorn app:app --host 0.0.0.0 --port 8000 --log-level warning &
```

- Use `reload=False` (default). `reload=True` causes issues with the lifespan cache.
- Access at http://localhost:8000
- Check cache status: `curl -s http://localhost:8000/api/status`
- Trigger manual refresh: `curl -s -X POST http://localhost:8000/api/refresh`

## Architecture

The app is a FastAPI server with an in-memory cache. On startup, it fetches all Jira issues and Confluence pages in parallel, caches them in RAM, and all searches run against that local cache. The cache auto-refreshes every 1 hour.

```
app.py               — FastAPI server, cache lifecycle, API endpoints
search.py            — Data fetching (Jira, Confluence, Drive) + local search logic
static/hub.html      — Single-page UI (dark theme, no build step)
static/hub.css       — UI 스타일시트
docs/presentation.html — QA Hub 팀 소개 발표 슬라이드 (/presentation 라우트)
.env                 — Credentials (ATLASSIAN_DOMAIN, ATLASSIAN_EMAIL, ATLASSIAN_API_TOKEN, JIRA_PROJECT, MCP_SSE_URL, REPOB_BIN)
```

### Cache flow
- `_load_cache()` runs `fetch_all_jira()` and `fetch_all_confluence()` concurrently via `asyncio.gather` + `ThreadPoolExecutor`
- Each source updates `CACHE["jira"]` / `CACHE["confluence"]` as soon as it completes (not after both finish)
- Internal fields prefixed with `_` (e.g. `_search_text`, `_desc_full`) are stripped from API responses by `clean()`

### Jira fetching
- Uses `/rest/api/3/search/jql` (NOT the deprecated `/rest/api/3/search`)
- Paginates via `nextPageToken` / `isLast` (NOT `startAt` / `total`)
- JQL: `project = GS AND updated >= -730d ORDER BY updated DESC`
- Jira description is in Atlassian Document Format (ADF); `_extract_adf_text()` recursively extracts plain text

### Confluence fetching
- Uses `/wiki/rest/api/content?spaceKey=GM` and `?spaceKey=CVS` separately
- **Do NOT use `/wiki/rest/api/content/search`** — its `start` pagination is broken (always returns the same first page regardless of offset)
- Fetches metadata only (no `body.view`) for speed; search is title-based
- `_search_text` field = lowercased title only

### Google Drive search
- Real-time (not cached); called on every search request
- **GWS CLI subprocess** (`drive_search_mcp` in `search.py`) — MCP SSE 방식 제거됨
- Query format: `name contains 'keyword'` (Google Drive API query syntax)

### GS OS (게임 온톨로지)
- `call_mcp_tool(tool_name, arguments)` in `search.py` — 인터페이스는 동일, 내부 구현 변경됨
- **게임 목록**: GS OS REST API (`GS_OS_API_URL/api/games`) → `_gs_os_games` 인메모리 캐시
- **CLI 도구** (`gs-os`): 환경변수 `GS_OS_SERVER_URL` + `NODE_TLS_REJECT_UNAUTHORIZED=0`
  - `search_games(query)` → `gs-os search <query>` (자연어/태그, 한국어는 빈 결과)
  - `get_game(game_name)` → REST 캐시에서 game_id 조회 후 `gs-os get <id>`
  - `similar_games` → `gs-os similar <id>`
  - `portfolio_stats` → `gs-os stats`
  - `get_dictionary` → `gs-os dict [tag]`
- 서버 시작 시 전체 게임 목록 캐싱 (`_get_gs_os_games()`)
- Results tagged with `"from_ontology": True` and merged with Drive results (ontology results take priority)

### Local search
- `search_jira_local()`: searches `summary + _desc_full` (title + full description)
- `search_confluence_local()`: searches `_search_text` (title only)
- Both support synonym expansion via `RELATED` dict in `search.py`
- Multi-word queries require ALL words to match (AND logic), each word can match via any synonym

### Synonym map
Add synonyms to `RELATED` dict in `search.py`. Each entry maps a word to its synonyms; add the reverse mapping too. Examples already defined: 잭팟↔jackpot↔jp, 쿠도↔kudo↔kudos, 버그↔bug, 크래시↔crash, etc.

### OpenAI GPT API
- Model: `gpt-4o-mini`
- API key: `OPENAI_API_KEY` in `.env`
- Used in two places:
  - `search.py` — `_gpt_translate()`: 검색어 한↔영 번역 (동의어 사전에 없는 단어 fallback), 결과는 `_TRANS_CACHE`에 메모이제이션
  - `app.py` — `_process_chat()`: 채팅 메시지에서 검색 키워드 추출 및 의도 분석

### QA 시트 자동 조회 (`_autofill_sheet_id`)
- `api/schedule` 호출 시 `qa_sheet_id`가 없는 항목 자동 조회
- `sheet_searched=1`이어도 `qa_start <= today`이면 **1시간 주기 재시도** (`_SHEET_RETRY_TS` dict로 스로틀)
- 재시도 이유: 시트는 QA 시작일 무렵 생성되므로, 등록 시점(이전)에 검색 실패해도 나중에 찾을 수 있음
- 일반 게임: MCP `get_game` 우선 → Drive 검색 fallback / SB 게임: Drive 검색 + "SB" 키워드

### Game panel document links (`/api/game_links`)
- Returns TC / GDD / MATH / CTD / Sound / 연출 links for the game side panel
- **GDD/MATH**: real-time Drive folder search — `_search_keywords()` tries tc_prefix → apostrophe-split longest part → `&→and` → raw name
- **Sound**: cached tab map (`sound_tabs`), matched by `tc_prefix`; auto-resolved from `sheet_games` if missing
- **연출**: cached tab map (`direction_tabs`), fuzzy match via `_norm_tab()` (strips apostrophes, hyphens, normalizes `&`/case)
- **CTD**: cached `ctd_game_info` rows, matched by game_id → game_name → tc_code
- Drive folders may contain subfolders — `_drive_search_in_folders()` returns folder URL first (subfolder structure support)
- Full design doc: `docs/game-links.md`
- Folder/sheet constants at top of `app.py` (~line 52): `_SOUND_SHEET_ID`, `_DIRECTION_SHEET_ID`, `_GDD_FOLDER_IDS`, `_MATH_FOLDER_ID`, `_CTD_GAME_INFO_GID`

## Key constraints

- Python 3.9 — use `Optional[X]` / `List[X]` from `typing`, not `X | None` or `list[X]`
- Confluence space keys: `GM` (Game Studio), `CVS` — not space names in CQL
- Jira JQL `-2y` not supported by new search/jql API; use `-730d` instead

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HyewonKim-GS) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-16 -->
