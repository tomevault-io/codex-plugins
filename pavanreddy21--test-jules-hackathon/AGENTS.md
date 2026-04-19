# Copilot Instructions: Canvas with LlamaIndex & Composio

## Architecture Overview

This is a **bidirectional AI canvas application** with real-time state synchronization between:
- **Frontend**: Next.js 15 + React 19 + CopilotKit (TypeScript)
- **Backend**: LlamaIndex AG-UI agent + FastAPI (Python)
- **External Integration**: Google Sheets via Composio API

The architecture uses **shared state as ground truth** - all changes flow through a centralized state object synchronized between UI and agent.

## Key Architectural Patterns

**⚠️ CRITICAL**: As you plan or implement any architectural changes, you MUST update this section to reflect the new reality.

### 1. State Synchronization via CopilotKit

The entire application state flows through `useCoAgent` hook in `src/app/page.tsx`:

```typescript
const { state, setState } = useCoAgent<AgentState>({
  name: "sample_agent",
  initialState,
});
```

**Critical Rule**: Never manipulate state directly. Always use `setState` or exposed Copilot actions. The LlamaIndex agent receives this state on every turn and can call frontend actions to mutate it.

### 2. Dual Tool System

**Frontend Actions** (defined with `useCopilotAction` in `page.tsx`):
- Exposed to the agent as callable tools
- Execute in the browser, directly mutating canvas state
- Examples: `createItem`, `setProjectField1`, `addProjectChecklistItem`
- Naming convention: `set{Type}Field{Number}` for field updates

**Backend Tools** (defined in `agent/agent.py`):
- Execute on the Python agent server
- Used for external integrations (Composio/Google Sheets)
- Examples: Composio tools loaded via `COMPOSIO_TOOL_IDS` env var

### 3. Card Type System

Four card types with **strict field schemas** (see `FIELD_SCHEMA` in `agent.py`):

- **project**: `field1` (text), `field2` (select), `field3` (date YYYY-MM-DD), `field4` (checklist array)
- **entity**: `field1` (text), `field2` (select), `field3` (string[] tags), `field3_options` (available tags)
- **note**: `field1` (textarea content)
- **chart**: `field1` (array of `{id, label, value}` where value is 0-100 or empty)

**Key Pattern**: Field numbers are semantic, not arbitrary. Always use the correct field action for the type.

### 4. Google Sheets Integration Architecture

**One-way sync**: Canvas → Sheets (automatic via `useEffect` debounce in `page.tsx`)
**Import flow**: Sheets → Canvas (via frontend API routes → Python agent)

Flow: Frontend `/api/sheets/sync` → Python `/sync-to-sheets` → Composio tools

The Python agent (`sheets_integration.py`) handles intelligent type detection when importing from sheets.

## Critical Development Workflows

### Running the Application

```bash
# Install all dependencies (Next.js + Python agent)
pnpm install && pnpm install:agent

# Start both servers concurrently
pnpm dev
# OR debug mode with verbose logging
pnpm dev:debug

# Run individually
pnpm dev:ui    # Next.js on :3000
pnpm dev:agent # FastAPI on :9000
```

**Port Dependencies**: Frontend expects agent at `http://127.0.0.1:9000` (hardcoded in `src/app/api/copilotkit/route.ts`)

### Environment Setup

Two separate `.env` files required:

**Root `.env.local`** (optional):
```bash
COPILOT_CLOUD_PUBLIC_API_KEY="" # For CopilotKit Cloud features
```

**`agent/.env`** (required):
```bash
OPENAI_API_KEY=""  # Required for LLM
COMPOSIO_API_KEY="" # For external tools
COMPOSIO_USER_ID="default" # User scope for Composio
COMPOSIO_TOOL_IDS="" # Comma-separated tool slugs (e.g., "GOOGLESHEETS_*")
```

### Adding New Card Types

1. Define type in `src/lib/canvas/types.ts` (add to `CardType` union and create data interface)
2. Update `defaultDataFor()` in `src/lib/canvas/state.ts`
3. Add renderer in `src/components/canvas/CardRenderer.tsx`
4. Update `FIELD_SCHEMA` constant in `agent/agent.py`
5. Create frontend actions in `src/app/page.tsx` following naming pattern `set{Type}Field{N}`
6. Add corresponding stub functions in `agent/agent.py` (frontend tool stubs section)

### Modifying Existing Fields

**Frontend**: Edit field rendering in `CardRenderer.tsx` and corresponding action handlers in `page.tsx`
**Backend**: Update `FIELD_SCHEMA` in `agent.py` - this is the agent's authoritative reference

## Project-Specific Conventions

### Field Numbering System

Fields use **sequential numbering** (`field1`, `field2`, `field3`, `field4`) rather than semantic names. This is by design for schema flexibility:

- Project `field1` = description text
- Project `field2` = select dropdown  
- Project `field3` = date picker
- Project `field4` = checklist array

**Never** use `field1` for non-note card subtitles/descriptions - use `setItemSubtitleOrDescription` instead.

### Idempotency Patterns

The codebase implements **aggressive deduplication** to prevent AI agent loops:

1. **Creation throttling** (`lastCreationRef` in `page.tsx`): Tracks recent creations with 5-second window
2. **Name-based deduplication**: `createItem` checks for existing items with same type+name
3. **Checklist/metric deduplication**: Sub-item creation returns existing ID if duplicate detected

### Human-in-the-Loop (HITL)

Implemented via **`renderAndWaitForResponse`** in frontend actions (not backend):

```typescript
useCopilotAction({
  name: "choose_item",
  renderAndWaitForResponse: ({ respond, args }) => {
    // Render UI, then call respond(value)
  }
});
```

Used for disambiguation when agent can't determine target (e.g., "which item to update?")

### Date Handling

Dates accept **natural language input** but normalize to `YYYY-MM-DD`:

```typescript
// Handler in setProjectField3 normalizes:
"tomorrow" → "2025-09-27"
"2025-01-30" → "2025-01-30"
Date object → "YYYY-MM-DD"
```

### State Caching Workaround

`cachedStateRef` in `page.tsx` prevents flicker during agent updates. This is a temporary fix for transient empty states during sync. **Do not remove** without addressing root cause in CopilotKit.

## Integration Points

### CopilotKit Runtime

Configured in `src/app/api/copilotkit/route.ts`:
- Uses `LlamaIndexAgent` adapter pointing to Python agent
- Agent name must match `useCoAgent({ name: "sample_agent" })`

### Composio Tools

**Dynamic loading** in `agent/agent.py` via `_load_composio_tools()`:
- Reads `COMPOSIO_TOOL_IDS` env var (comma-separated slugs)
- Requires `COMPOSIO_API_KEY` and `COMPOSIO_USER_ID`
- Tools are LlamaIndex-compatible via `LlamaIndexProvider()`

### Google Sheets API Routes

All at `/api/sheets/*`:
- `/sync` - Import from sheets (GET sheet data)
- `/create` - Create new spreadsheet
- `/list` - Get available sheet names
- Backend handles actual Composio API calls in `agent/agent/server.py`

## Common Gotchas

1. **Agent "I don't have access to that tool"**: Check that frontend actions are registered with `useCopilotAction` and `available: "remote"`
2. **State not syncing**: Verify shared state field names match between TypeScript types and Python agent's `initial_state`
3. **Infinite loops**: Agent repeatedly calling same action - add throttling or check `lastAction` in state
4. **Checklist item IDs**: Accept both string ID (e.g., "001") and numeric index (1-based or 0-based) for flexibility
5. **Lock files ignored**: `.gitignore` excludes all lock files to support multiple package managers - regenerate yours after clone

## Testing & Debugging

**Enable debug logging**:
```bash
pnpm dev:debug  # Sets LOG_LEVEL=debug
```

**Check agent connectivity**: If UI shows "trouble connecting", verify:
1. Python agent running on port 9000
2. `OPENAI_API_KEY` set in `agent/.env`
3. Both servers started successfully

**State inspection**: Use JSON view toggle (button at bottom) to see raw state structure

**Kill stuck processes**:
```bash
lsof -ti:9000 | xargs kill -9  # Agent port
lsof -ti:3000 | xargs kill -9  # UI port
```

## Stack-Specific Notes

- **Package manager agnostic**: Use pnpm/npm/yarn/bun (lock files not committed)
- **Python environment**: `uv` manages `.venv` in `agent/` directory (auto-created on `pnpm install:agent`)
- **Tailwind + shadcn/ui**: Component library in `src/components/ui/` - use existing components, don't create custom ones
- **Turbopack enabled**: `next dev --turbopack` for faster dev builds
- **No middleware**: All routing via App Router conventions

---
> Source: [pavanreddy21/test-jules-hackathon](https://github.com/pavanreddy21/test-jules-hackathon) — distributed by [TomeVault](https://tomevault.io/claim/pavanreddy21).
<!-- tomevault:4.0:agents_md:2026-04-17 -->
