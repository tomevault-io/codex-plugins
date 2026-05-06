# Browser Testing Procedure

After editing any file under `extension/`, test changes using Playwright MCP as the primary tool.

## Step 1: Reload the extension

Run from the project root after any edit to `extension/` files:

```
node dev/test.mjs setup
```

This reloads the extension, refreshes the AID tab (for fresh token), closes stale popup tabs, opens a new popup tab, and polls until it shows scenario data. Output includes JSON status.

- Manifest changes require a manual `chrome://extensions` reload first

## Step 2: Test with Playwright MCP (primary)

Playwright MCP tools are the primary way to inspect and interact with the popup.

### Inspect the popup

1. `browser_tabs` (action: `list`) — find the popup tab
2. `browser_tabs` (action: `select`, index) — switch to the popup tab
3. `browser_snapshot` — read the full a11y tree (structure, text, element refs)

### Interact with the popup

- `browser_click` — click buttons, branches, links, toggles
- `browser_type` / `browser_fill_form` — fill text inputs, textareas, selects
- `browser_take_screenshot` — visual verification when a11y tree isn't enough
- `browser_wait_for` — wait for text to appear/disappear after actions

### Typical Playwright test cycle

1. Edit extension files
2. `node dev/test.mjs setup` — reload + open popup + verify loaded
3. `browser_tabs list` → `browser_tabs select` (popup tab)
4. `browser_snapshot` — read current state
5. `browser_click` / `browser_type` — perform actions
6. `browser_snapshot` — verify result

### Popup tab access via Playwright code

If `browser_tabs` doesn't show the popup, create one via CDP:

```js
async (page) => {
  const client = await page.context().newCDPSession(page);
  await client.send('Target.createTarget', {
    url: 'chrome-extension://leahjenbcfenoojjdggddjldhiblkbje/popup/popup.html'
  });
  return 'Popup tab created';
}
```

Then select it via `browser_tabs`.

## Supplementary: `dev/test.mjs` commands

Use the CDP test harness for things Playwright can't do directly:

| Command | Use when... |
|---------|-------------|
| `setup` | Reloading extension + opening popup (always run first) |
| `status` | Quick JSON check: token, scenario, branchCount, panel visibility |
| `branches` | List branch names and shortIds |
| `eval <expr>` | Run JS in popup context (access `window.ctx`, call `openClonePanel()`, etc.) |
| `run <file.js>` | Execute a JS file in popup context (no shell quoting issues) |
| `sw-eval <expr>` | Run JS in service worker context |
| `sw-log` | Stream service worker console (debug background issues) |
| `popup-log` | Stream popup tab console (debug popup issues) |
| `snap` | Text dump of popup (quick check without Playwright) |

## Rules

- **Playwright MCP is primary** — use `browser_snapshot` + `browser_click` for all UI testing
- **`test.mjs` is supplementary** — use for reload, service worker access, console streaming, and JS eval
- **ALWAYS** run `node dev/test.mjs setup` after editing extension files (handles reload + popup creation)
- **NEVER** call `browser_console_messages` on AID pages — too noisy, causes agent stalls
- **NEVER** use `browser_navigate` or `page.goto()` to AID URLs — crashes MCP
- Use `sw-log` to debug service worker issues instead of guessing
- Extension ID is stable: `leahjenbcfenoojjdggddjldhiblkbje` (hardcoded key in manifest)

---
> Source: [LewdLeah/Multiple-Choice-Assistant](https://github.com/LewdLeah/Multiple-Choice-Assistant) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-02 -->
