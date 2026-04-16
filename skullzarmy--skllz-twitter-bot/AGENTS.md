# AI Agent Instructions for SKLLZ Twitter Bot

## Project Overview

This is a **Bun/TypeScript Twitter bot** for NFT artists on Tezos. It automates:

1. **Thank-you tweets** when art sells (synced from objkt.com + TzKT APIs)
2. **Weekly shill threads** showcasing latest 5 artworks
3. Uses **OpenAI o3-mini** for AI-generated tweet content
4. Uses **PostgreSQL** for tracking sales, tokens, and dynamic schedules

### Supported NFT Types

The bot currently tracks and promotes three types of Tezos NFTs:

1. **Standard Tokens (1/1s)** - Unique pieces with active listings on objkt.com
   - Fetched via `getMyMintedTokens()` in `objkt.ts`
   - Includes editions with limited supply

2. **Open Editions** - Unlimited mints available for purchase
   - Fetched via `getActiveOpenEditions()` in `objkt.ts`
   - Tracks individual mints as sales events

3. **Generative Collections (editart)** - Algorithmic art collections
   - Fetched via `getGenerativeCollections()` in `objkt.ts`
   - Each minted token tracked individually
   - Supports editart-style generative projects

All three types sync to the same `tokens` table and generate thank-you tweets when sold/minted.

## Critical Architecture Points

### 1. Bun Runtime & ESM Modules

- **Always use ESM syntax**: `import`/`export`, not CommonJS
- Entry points use `import.meta.main` pattern (see `thank-you.ts`, `shill-thread.ts`)
- Scripts run via `bun run <file>` (no transpilation needed)
- Use `process.argv.includes('--flag')` for CLI args

### 2. Database-Driven Scheduling (Not Simple Cron)

- `src/index.ts` loads schedules from PostgreSQL `schedules` table
- Uses `croner` library to create dynamic jobs at runtime
- Schedules managed via CLI: `bun run schedule add/list/enable/disable/remove`
- **PostgreSQL advisory locks** (`utils/locks.ts`) prevent duplicate runs of same schedule
- Jobs call `executeSchedule()` which wraps `processThankYouTweets()` or `processShillThread()`

### 3. Dual Workflow Pattern: Direct CLI + Scheduled Execution

Every automation has 2 entry points:

- **Direct**: `bun run thank` or `bun run shill` (calls exported function)
- **Scheduled**: `src/index.ts` calls same exported function via schedule trigger
- Both support `--dry-run` flag for testing without posting tweets

### 4. Tezos NFT Data Pipeline

Data flows through 3 layers:

1. **Fetch** (`objkt.ts`): GraphQL queries to objkt.com
   - `getMyMintedTokens()` - 1/1s with active listings
   - `getActiveOpenEditions()` - open editions
   - `getGenerativeCollections()` - generative collections
   - `getListingSales()` / `getOpenEditionSales()` / `getGenerativeSales()` - sales data
2. **Transform** (`transformers.ts`): Convert GraphQL responses to flat structures
   - `transformMintedToken()`, `transformOpenEdition()`, `transformGenerativeCollection()`
   - `transformListingSale()`, `transformMintSale()` - generate sale records
3. **Sync** (`sync.ts`): Upsert to PostgreSQL `tokens` and `nft_sales` tables
   - `syncObjktData()` - called before every tweet automation run
   - Handles duplicates via `ON CONFLICT` clauses

### 5. OpenAI Integration (o3-mini)

- **Model requirement**: Use `o3-mini` (not gpt-4o) for reasoning capabilities
- **Token parameter**: Use `max_completion_tokens` (not `max_tokens`)
- All prompts centralized in `src/prompts.ts` with `system` and `user` messages
- Generates authentic, varied tweets (anti-spam pattern)

### 6. Twitter API v2 Usage

- Library: `twitter-api-v2` (OAuth 1.0a)
- Client created via `createTwitterClient()` in `index.ts`
- Thread posting pattern in `shill-thread.ts`:
  - Post intro tweet → capture `tweet.data.id`
  - Reply to previous tweet using `{ reply: { in_reply_to_tweet_id } }`
  - 2-3s random delay between tweets to avoid rate limits
  - Retry with exponential backoff (`retryWithBackoff()`)

## Key Conventions

### Configuration

- All env vars loaded via `config.ts` (uses `dotenv/config`)
- Validation: `getEnvVar()` throws if required var missing
- Wallet addresses are **comma-separated** in `WALLET_ADDRESSES` env var

### Database Access

- Single global `pool` exported from `db.ts`
- Always parameterize queries: `pool.query('SELECT * FROM tokens WHERE id = $1', [id])`
- Dry-run pattern: Log instead of executing when `dryRun = true`

### Logging

- Use `logger` from `utils/logger.ts` (not `console.log`)
- Emoji prefixes: `✅` success, `❌` errors, `⚠️` warnings, `🔍` dry-run, `🐦` tweets

### Error Handling

- Graceful shutdown hooks in `index.ts`: `SIGINT`/`SIGTERM` → release locks + close DB
- Unhandled rejections/exceptions logged before exit
- Retry logic with backoff for Twitter API calls

## Common Development Workflows

### Testing Changes

```bash
# Test API connections
bun run test

# Dry-run thank-you tweets (no posts)
bun run thank:dry-run

# Dry-run shill thread
bun run shill:dry-run

# Test data sync
bun run sync:dry-run
```

### Adding New Features

#### New Tweet Type

1. Add prompt templates to `src/prompts.ts`:
   ```typescript
   export const prompts = {
     myNewTweet: {
       system: `System prompt defining AI behavior...`,
       user: (param1, param2) => `User prompt with ${param1}...`,
     },
   };
   ```
2. Create new file `src/my-new-tweet.ts` following the dual-workflow pattern:
   - Export main function: `export async function processMyNewTweet(dryRun = false)`
   - Add CLI entry point: `if (import.meta.main) { ... }`
   - Add dry-run support throughout
3. Add npm script to `package.json`:
   ```json
   "my-tweet": "bun run src/my-new-tweet.ts",
   "my-tweet:dry-run": "bun run src/my-new-tweet.ts --dry-run"
   ```
4. Update `Schedule['type']` union in `src/index.ts` and `src/schedule.ts`
5. Add case to `executeSchedule()` in `src/index.ts`

#### New GraphQL Query/Data Source

1. Add query to `src/objkt.ts`:
   ```typescript
   export async function getMyNewData(
     client: GraphQLClient,
     wallets: string[]
   ) {
     const query = `query { ... }`;
     return await client.request(query, { wallets });
   }
   ```
2. Create transformer in `src/transformers.ts`:
   ```typescript
   export function transformMyData(raw: any): CleanData {
     return {
       /* flatten nested structure */
     };
   }
   ```
3. Add sync logic to `src/sync.ts`:
   - Call fetch function
   - Transform data
   - Upsert to database (use `ON CONFLICT` for idempotency)
   - Add to `syncObjktData()` function

#### New AI Prompt

1. Edit `src/prompts.ts` - add to existing object or create new category
2. Keep tweets ≤ 280 characters in system prompt
3. Use template literal functions for dynamic content: `user: (name, url) => \`...\``
4. Test with dry-run: `bun run <tweet-type>:dry-run`
5. Iterate based on output quality and character count

### Database Migrations

- Run: `bun run migrate` (executes `migrations/001_create_schedules.sql`)
- Schema: `tokens`, `nft_sales`, `schedules` tables
- Use `ON CONFLICT` for idempotent upserts

### Editing AI Prompts

1. Edit `src/prompts.ts` (system/user message templates)
2. Test with dry-run: `bun run thank:dry-run`
3. Verify output ≤ 280 chars
4. Commit changes to version control

## Common Pitfalls

1. **Don't use `gpt-4o`** → Use `o3-mini` model (configured in `OPENAI_MODEL`)
2. **Don't forget `--dry-run` flag** when testing to avoid accidental tweets
3. **PostgreSQL advisory locks use schedule ID** → Must match across `acquireLock(scheduleId)` calls
4. **Twitter handles extracted from URLs** → `cleanTwitterHandle()` in `thank-you.ts` converts `https://x.com/user` to `@user`
5. **Sale batching by token name** → Multiple buyers for same artwork grouped in `batchSalesByToken()`
6. **Mint sales need synthetic ID** → Hash `fa_contract:token_id` for unique `sale_id` in `insertMintSale()`

## File Organization

```
src/
├── index.ts           # Main scheduler + graceful shutdown
├── config.ts          # Env var loading & validation
├── db.ts              # PostgreSQL pool singleton
├── sync.ts            # Objkt/TzKT data sync pipeline
├── thank-you.ts       # Thank-you tweet automation
├── shill-thread.ts    # Weekly shill thread automation
├── schedule.ts        # Schedule management CLI
├── prompts.ts         # ⚠️ AI prompt templates (customize here)
├── openai.ts          # OpenAI client factory
├── objkt.ts           # Objkt GraphQL queries
├── tzkt.ts            # TzKT REST API calls
├── transformers.ts    # Data transformation layer
└── utils/
    ├── logger.ts      # Structured logging
    └── locks.ts       # PostgreSQL advisory locks
```

## Quick Reference Commands

```bash
# Development
bun run dev              # Auto-reload on file changes
bun run start            # Production scheduler
bun run test             # Test all API connections

# Data Management
bun run sync             # Sync NFT data from blockchain
bun run mark-processed   # Mark all sales as processed (one-time setup)

# Automation (dry-run recommended first)
bun run thank:dry-run    # Preview thank-you tweets
bun run thank            # Post thank-you tweets
bun run shill:dry-run    # Preview shill thread
bun run shill            # Post shill thread

# Schedule Management
bun run schedule add thank "0 * * * *" UTC     # Hourly thank-yous
bun run schedule add shill "0 12 * * 2" UTC    # Tuesdays at noon
bun run schedule list                           # Show all schedules
bun run schedule enable/disable/remove <id>     # Manage schedules
```

## When Editing Code

- **TypeScript strict mode enabled**: Handle nulls, use typed interfaces
- **Prefer transactions for multi-step DB operations** (currently missing in some places)
- **Add dry-run support** to any new automation functions
- **Export main function** for both CLI (`import.meta.main`) and scheduler imports
- **Test with real Twitter/OpenAI APIs** in dry-run mode before production

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/skullzarmy) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
