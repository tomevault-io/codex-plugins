
# League Client MCP

You are working with a **live League of Legends Client** via the `league-client-mcp` MCP server.
The server bridges tool calls to a Pengu Loader plugin inside the LoL Client over WebSocket.

## Architecture

```
Windsurf Cascade  ──MCP stdio──►  MCP Server (Node.js)
                                      │  ws://127.0.0.1:8080
                                      ▼
                           Pengu Loader Plugin  (LoL Client)
                                      │  DOM · CSS · JS · LCU
                                      ▼
                           League of Legends Client  (CEF / Ember.js)
```

League must be running with Pengu Loader active. 16 MCP tools available.

## Standard Workflow

```
1. get_lol_dom_snapshot   → map the current page (find selectors)
2. inject_lol_css         → prototype CSS changes instantly
3. query_lol_element      → verify a specific selector
4. inject_lol_plugin      → add persistent JS + CSS
5. reload_lol_plugin      → iterate without full restart
6. export_plugin_to_pengu → write to disk
7. reload_lol_client      → activate permanently
```

## Tool Quick Reference

| Tool | Purpose |
|---|---|
| `get_lol_dom_snapshot` | Snapshot current page HTML |
| `query_lol_element` | Inspect one element (style, rect, attrs) |
| `inject_lol_css` | Inject/replace global CSS |
| `execute_lol_javascript` | Run one-off JS snippet |
| `inject_lol_plugin` | Persistent named plugin (JS + CSS) |
| `remove_lol_plugin` | Remove and clean up plugin |
| `reload_lol_plugin` | Teardown + re-execute plugin |
| `export_plugin_to_pengu` | Write plugin file to disk |
| `lcu_request` | HTTP to LCU REST API |
| `click_lol_element` | Click element by selector |
| `type_into_lol_element` | Type into input |
| `wait_for_lol_element` | Wait for element after navigation |
| `get_lol_client_state` | Current URL and active plugins |
| `get_lol_performance_metrics` | Heap, DOM nodes, paint timings |
| `get_lol_screenshot` | Capture LeagueClientUx to temp PNG |
| `reload_lol_client` | Full client reload |

## Key LCU Endpoints

### Summoner
- `GET /lol-summoner/v1/current-summoner` — id, puuid, displayName, summonerLevel
- `GET /lol-summoner/v2/summoners/puuid/{puuid}` — summoner by PUUID
- `GET /lol-summoner/v1/current-summoner/rerollPoints` — ARAM reroll points

### Gameflow
- `GET /lol-gameflow/v1/gameflow-phase` — `None`·`Lobby`·`ChampSelect`·`InProgress`·`EndOfGame`
- `GET /lol-gameflow/v1/session` — full session data

### Champion Select
- `GET /lol-champ-select/v1/session` — picks, bans, timer, actions
- `PATCH /lol-champ-select/v1/session/actions/{id}` — hover/lock a champion
- `POST /lol-champ-select/v1/session/actions/{id}/complete` — confirm action
- `GET /lol-champ-select/v1/session/my-selection` — my champion and spells
- `PATCH /lol-champ-select/v1/session/my-selection` — update summoner spells
- `GET /lol-champ-select/v1/pickable-champion-ids` — available picks
- `GET /lol-champ-select/v1/bannable-champion-ids` — available bans
- `GET /lol-champ-select/v1/session/timer` — phase and time remaining
- `GET /lol-champ-select/v1/all-grid-champions` — all champions with ownership

### Lobby
- `GET /lol-lobby/v2/lobby` — current lobby (gameConfig, members)
- `POST /lol-lobby/v2/lobby` — create lobby `{queueId}`
- `DELETE /lol-lobby/v2/lobby` — leave/destroy
- `POST /lol-lobby/v2/lobby/matchmaking/search` — start queue
- `DELETE /lol-lobby/v2/lobby/matchmaking/search` — cancel queue
- `GET /lol-lobby/v2/received-invitations` — pending invitations
- `POST /lol-lobby/v2/received-invitations/{id}/accept` — accept
- `POST /lol-lobby/v2/lobby/members/{summonerId}/kick` — kick member
- `PUT /lol-lobby/v2/lobby/members/localMember/position-preferences` — set lanes

### Chat & Social
- `GET /lol-chat/v1/friends` — friends with availability and game status
- `GET /lol-chat/v1/friend-counts` — online/total counts
- `GET /lol-chat/v1/me` — my chat state
- `PUT /lol-chat/v1/me` — update status / availability (`chat`·`away`·`offline`)
- `GET /lol-chat/v1/conversations` — active conversations
- `POST /lol-chat/v1/conversations/{id}/messages` — send message
- `POST /lol-chat/v2/friend-requests` — send friend request

### Ranked
- `GET /lol-ranked/v1/current-ranked-stats` — tier, division, LP, wins, losses
- `GET /lol-ranked/v1/ranked-stats/{puuid}` — any player's ranked stats
- `GET /lol-ranked/v1/current-lp-change-notification` — LP change from last game

### Match History
- `GET /lol-match-history/v1/products/lol/current-summoner/matches` — recent matches
- `GET /lol-match-history/v1/games/{gameId}` — detailed game data
- `GET /lol-match-history/v1/recently-played-summoners` — recent teammates

### Champion Mastery
- `GET /lol-champion-mastery/v1/local-player/champion-mastery` — all mastery data
- `GET /lol-champion-mastery/v1/local-player/champion-mastery-score` — total score

### Runes
- `GET /lol-perks/v1/pages` — all rune pages
- `POST /lol-perks/v1/pages` — create rune page
- `GET /lol-perks/v1/currentpage` — active page
- `PUT /lol-perks/v1/currentpage` — set active (body: `{id}`)
- `GET /lol-perks/v1/recommended-pages/champion/{championId}/position/{position}/map/{mapId}` — recommended

### Loot & Inventory
- `GET /lol-loot/v1/player-loot` — loot inventory
- `POST /lol-loot/v1/recipes/{recipeName}/craft` — craft loot
- `GET /lol-inventory/v1/wallet` — wallet (RP, BE)
- `GET /lol-champions/v1/inventories/{summonerId}/champions` — owned champions
- `GET /lol-champions/v1/inventories/{summonerId}/champions/{championId}/skins` — owned skins

### Honor & End of Game
- `GET /lol-honor-v2/v1/ballot` — post-game honor ballot
- `POST /lol-honor-v2/v1/honor-player` — honor player
- `GET /lol-honor-v2/v1/profile` — honor level
- `GET /lol-end-of-game/v1/eog-stats-block` — post-game stats

### Missions & Clash
- `GET /lol-missions/v1/missions` — active missions
- `GET /lol-clash/v1/player` — Clash eligibility
- `GET /lol-clash/v1/tournament-summary` — active tournaments

## Plugin Code Conventions

```js
// Minimal plugin template
const obs = new MutationObserver(() => {
  clearTimeout(window._t);
  window._t = setTimeout(render, 350);
});
obs.observe(document.body, { childList: true, subtree: false });
const interval = setInterval(render, 5000);
render();

// REQUIRED: return cleanup
return () => {
  obs.disconnect();
  clearInterval(interval);
  clearTimeout(window._t);
  document.getElementById('my-plugin')?.remove();
};
```

- **Always return `() => void`** — required for reload and remove
- Debounce MutationObserver ~350 ms for SPA navigation
- `fetch('/lcu-endpoint')` works inside plugins (no auth needed)
- `DataStore.set('k', v)` / `DataStore.get('k')` for persistence

## CSS Conventions

- `!important` on every declaration
- `position: fixed; z-index: 99999+` for overlays
- Design tokens: `--font-display`, `--font-body`, `--color-gold-{1-5}`, `--color-blue-{1-6}`, `--color-grey-{1-6}`
- Common selectors: `.lol-navigation`, `.screen-root.active`, `.profile-wrapper`, `.champion-select`

## Constraints

- CEF/Chromium — ES2020+; no Node.js or browser extension APIs  
- `inject_lol_css` replaces entire stylesheet on each call
- Injected plugins are cleared on reload — `export_plugin_to_pengu` makes them permanent
- Ember.js DOM is async — debounce reads or use `wait_for_lol_element`
- 15 s WebSocket timeout if LoL Client or Pengu Loader is not running

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rumi-chan)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/rumi-chan)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
