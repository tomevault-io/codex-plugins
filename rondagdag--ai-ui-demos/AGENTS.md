# AI UI Demos — Copilot Instructions

## Architecture Overview

This is an **educational demo** showcasing three AI UI patterns: CopilotKit, Vercel AI SDK Generative UI, and AG-UI protocol.

```
Browser → Next.js (port 3000) → OpenAI API
```

- **Frontend**: Next.js 16 + React 19 + TypeScript + Tailwind CSS
- **Backend**: Next.js API routes connecting to OpenAI API
- **Models**: OpenAI models (GPT-4o, GPT-4o-mini, etc.)

## Critical Developer Workflows

### Starting the Application

**Single command:**

```bash
npm run dev  # runs on port 3000
```

Open <http://localhost:3000> and pick a demo!

### Environment Setup

- Required env vars in `.env.local`:
  - `OPENAI_API_KEY` (get from <https://platform.openai.com/api-keys>)
  - `OPENAI_MODEL_NAME` (default: `gpt-4o`)

### Switching Models

Change `OPENAI_MODEL_NAME` in `.env.local`:
- `gpt-4o` → GPT-4o (recommended, best tool calling)
- `gpt-4o-mini` → GPT-4o mini (faster, cheaper)
- `gpt-4-turbo` → GPT-4 Turbo
- `gpt-3.5-turbo` → GPT-3.5 Turbo

## The Three Demo Patterns

### Demo 1: Trip Planner — CopilotKit Pattern

**Location**: [src/app/trip-planner/page.tsx](src/app/trip-planner/page.tsx)

**Key APIs**:
- `useCopilotReadable({ value, description })` — Share app state with AI
- `useCopilotAction({ name, description, parameters, handler })` — Register callable functions
- `CopilotPopup` — Pre-built chat UI component
- Backend: [src/app/api/copilotkit/route.ts](src/app/api/copilotkit/route.ts) uses `OpenAIAdapter` with `CopilotRuntime`

**Pattern**: AI reads state via `useCopilotReadable` and modifies app via `useCopilotAction` handlers. Actions update React state directly.

**Important**: CopilotKit connects directly to OpenAI API. Configure via `OPENAI_API_KEY` and `OPENAI_MODEL_NAME` env vars.

### Demo 2: Recipe Explorer — Generative UI Pattern

**Location**: [src/app/recipe-explorer/page.tsx](src/app/recipe-explorer/page.tsx)

**Key APIs**:
- `useChat` hook from `@ai-sdk/react` for streaming
- Server-side `streamText` with tool definitions ([src/app/api/chat/route.ts](src/app/api/chat/route.ts))
- Zod schemas for tool input validation
- `message.parts[]` contains both text and tool invocations

**Pattern**: AI calls tools that return structured data (instead of text). Client maps tool names to React components:
```typescript
if (part.type === "tool-showRecipe" && part.state === "output-available") {
  return <RecipeCard {...part.input} />;
}
```

**Critical**: Message transformation required in API route — useChat sends `parts[]`, but `streamText` expects `content`. Transform before passing to AI:
```typescript
const transformedMessages = messages.map(msg => {
  if (msg.parts) {
    const content = msg.parts.filter(p => p.type === 'text').map(p => p.text).join('\n');
    return { role: msg.role, content };
  }
  return msg;
});
```

### Demo 3: Story Builder — AG-UI Protocol Pattern

**Location**: [src/app/story-builder/page.tsx](src/app/story-builder/page.tsx)

**Key Concepts**:
- Raw SSE (Server-Sent Events) streaming via EventSource
- 17 typed AG-UI events: `RUN_STARTED`, `TEXT_MESSAGE_*`, `TOOL_CALL_*`, `STATE_DELTA`, `RUN_FINISHED`
- A2UI pattern: agent generates declarative component definitions via tool calls
- Backend: [src/app/api/agent/route.ts](src/app/api/agent/route.ts)

**Pattern**: Server emits typed events as SSE. Client processes events and renders components:
```typescript
RUN_STARTED → STATE_SNAPSHOT → TOOL_CALL_START → TOOL_CALL_ARGS → 
TOOL_CALL_END → STATE_DELTA → RUN_FINISHED
```

**Critical Bug Fix**: Demo 3 has a failsafe mechanism because LLMs sometimes skip required tool calls. If `show_choices` is not called, the backend makes a second LLM call specifically to generate choices, then streams the result.

**Gotcha**: Always parse `TOOL_CALL_ARGS` deltas incrementally — they arrive as character chunks, not complete JSON.

## Project-Specific Conventions

### File Organization

- API routes use **extensive inline documentation** — this is an educational codebase
- Each demo has a detailed header comment explaining the pattern
- Component files in `src/components/` are pure presentational (no AI logic)
- Shared model config in `src/lib/model.ts` creates OpenAI-compatible client

### Tool Calling Patterns

**All three demos use tool calling, but differently:**

| Demo | Tool Location | Tool Definition | Tool Result |
|------|--------------|-----------------|-------------|
| CopilotKit | Frontend (`useCopilotAction`) | Parameters array with types | Handler returns string message |
| Generative UI | Backend API route | Zod schemas | Returns structured data for component props |
| AG-UI | Backend API route | OpenAI tool format | Streamed as `TOOL_CALL_*` events |

**Zod validation example** (Generative UI):
```typescript
inputSchema: z.object({
  name: z.string().describe("Recipe name"),
  ingredients: z.array(z.string()).describe("List of ingredients"),
})
```

### Error Handling Philosophy

- Demo 1 (CopilotKit): Errors bubble up to CopilotKit's built-in error UI
- Demo 2 (Generative UI): Tool execute functions can return UI components on error
- Demo 3 (AG-UI): Emits `RUN_ERROR` event with message, includes fallback UI generation

## Common Pitfalls

1. **Missing API key**: All demos require `OPENAI_API_KEY` in `.env.local`. Get one from <https://platform.openai.com/api-keys>

2. **Missing message transformation**: useChat sends `parts[]`, but some endpoints expect `content`. Check [src/app/api/chat/route.ts#L20-L35](src/app/api/chat/route.ts) for the transformation pattern.

3. **Tool call timing in Demo 3**: The LLM may skip required tool calls (especially `show_choices`). The backend has a failsafe that makes a second LLM call — don't remove this logic.

4. **Array parameters in CopilotKit**: Arrays must be typed as `"object"` in `useCopilotAction` parameters, then handle both array and object formats in the handler (see [src/app/trip-planner/page.tsx#L91-L102](src/app/trip-planner/page.tsx)).

## Integration Points

- **OpenAI SDK**: Used in Demo 3 ([src/app/api/agent/route.ts](src/app/api/agent/route.ts)) for tool calling with streaming
- **Vercel AI SDK**: Used in Demo 2 ([src/app/api/chat/route.ts](src/app/api/chat/route.ts)) for `streamText` and tool definitions
- **CopilotKit Runtime**: Used in Demo 1 ([src/app/api/copilotkit/route.ts](src/app/api/copilotkit/route.ts)) with `OpenAIAdapter`
- **OpenAI API**: All demos connect to OpenAI's API for GPT-4o and other models

## When Adding New Features

**New CopilotKit action**: Add `useCopilotAction` hook in component, update system prompt in `CopilotPopup` instructions prop

**New Generative UI tool**: Add tool definition in [src/app/api/chat/route.ts](src/app/api/chat/route.ts), create component in `src/components/`, map `part.type` in [src/app/recipe-explorer/page.tsx](src/app/recipe-explorer/page.tsx)

**New AG-UI component**: Add tool to `agentTools` array in [src/app/api/agent/route.ts](src/app/api/agent/route.ts), update event processor in [src/app/story-builder/page.tsx](src/app/story-builder/page.tsx)

**New demo**: Follow the pattern of existing demos — route in `src/app/`, API route in `src/app/api/`, components in `src/components/`

---
> Source: [rondagdag/ai-ui-demos](https://github.com/rondagdag/ai-ui-demos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-26 -->
