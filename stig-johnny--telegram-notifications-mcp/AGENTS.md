# Copilot Code Review Instructions - telegram-notifications-mcp

## Project Overview

MCP server for sending notifications and collecting interactive responses via Telegram. Used by Claude Code to notify the user and ask questions when blocked or needing input.

## Architecture

- **Pattern:** MCP SDK handler pattern (stdio transport)
- **Runtime:** Node.js (CommonJS)
- **Dependencies:** `@modelcontextprotocol/sdk` (uses built-in `fetch()` for HTTP)
- **Entry point:** `index.js` (single-file server)
- **API:** Telegram Bot API via HTTPS

## Security Focus

- `TELEGRAM_BOT_TOKEN` must never be hardcoded - from env or config file
- `TELEGRAM_CHAT_ID` must never be hardcoded - from env or config file
- Config file path (`TELEGRAM_CONFIG_FILE`) should not be user-controllable
- Never log bot token (appears in API URLs)
- Message content may contain sensitive build info - avoid logging full messages
- Inline keyboard callback data should be validated before processing

## Code Patterns

### Telegram API
- Base URL: `https://api.telegram.org/bot{token}/`
- Use `sendMessage` with `parse_mode: "Markdown"` for formatted messages
- Use `InlineKeyboardMarkup` for interactive buttons (yes/no, choices)
- Poll for responses via `getUpdates` with `offset` for deduplication

### Interactive Responses
- `ask_user_telegram` - Send question, poll for text reply
- `ask_yes_no_telegram` - Inline keyboard with Yes/No buttons
- `ask_choice_telegram` - Inline keyboard with 2-4 option buttons
- `get_telegram_response` - Check for pending callback query answers

### Message Formatting
- Use Markdown formatting for readability
- Urgency levels: low (no prefix), normal (default), high (with alert emoji)
- Keep messages concise - Telegram has 4096 character limit per message

## Common Pitfalls

- Telegram Bot API rate limit: ~30 messages/second per chat
- `getUpdates` long polling blocks - use timeout parameter
- Callback queries must be answered with `answerCallbackQuery` or button stays loading
- Bot must be started by user first (`/start` command) before sending messages
- Markdown parsing is strict - unmatched `*` or `_` will cause API errors

---
> Source: [Stig-Johnny/telegram-notifications-mcp](https://github.com/Stig-Johnny/telegram-notifications-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-29 -->
