# AI Coding Agent Instructions for Mathematica AI Chat App

## Project Overview
This is a Next.js-based AI chat application called "Mathematica AI" that integrates with Google's Gemini AI for conversational interactions. It uses Supabase for authentication, database storage, and real-time features.

## Architecture & Data Flow

### Core Components
- **Frontend**: Next.js 15 with React 19, TypeScript, Tailwind CSS, and Shadcn UI components
- **Backend**: Next.js API routes handling chat interactions
- **Database**: Supabase PostgreSQL with custom tables for chats, messages, and user associations
- **AI Integration**: Google Gemini 2.5 Pro via Vercel AI SDK with streaming responses and reasoning capabilities

### Database Schema
Key tables and relationships:
- `chats`: Stores chat sessions with `chat_id` (UUID), `chat_name`, timestamps
- `messages`: Core message storage with `message_id` (UUID), `content`, `role` (user/assistant), `chat_id`
- `user_chats`: Links users to their chats (many-to-many)
- `chat_msgs`: Links chats to messages (many-to-many)
- `user_msgs`: Extended message metadata with `sender_type` (user/model) and `model_name`

All tables use CASCADE deletes to maintain referential integrity when chats or messages are removed.

### Authentication Flow
- Supabase Auth with Google OAuth enabled
- Server-side user verification in API routes
- Client-side Supabase client for real-time subscriptions

## Critical Workflows

### Local Development Setup
1. Start Supabase local services: `supabase start`
2. Run Next.js dev server: `npm run dev` (uses Turbopack for faster builds)
3. Access at http://localhost:3000

### Chat Interaction Flow
1. User submits prompt via `ChatPromptInput` component
2. API route `/api/chat` validates auth and chat existence
3. Streams AI response from Gemini with reasoning enabled
4. Stores both user message and AI response in database
5. Updates UI with streaming content and final database state

### AI Response Processing
AI responses contain structured content wrapped in tags:
```
<reasoning-start>
[Reasoning content with **Step Title** sections]
<reasoning-end>

<text-start>
[Main response content]
<text-end>
```

The `Response` component parses this format and renders reasoning steps as collapsible sections using `ChainOfThought` UI components.

## Project-Specific Conventions

### Component Organization
- `src/components/atom/`: Basic reusable components (buttons, avatars, etc.)
- `src/components/layout/`: Page layout components (chat body, sidebar, etc.)
- `src/components/ui/`: Shadcn UI components and custom variants
- Use `motion/react` for animations with spring transitions (stiffness: 400, damping: 20)

### Naming Patterns
- Database columns: snake_case (e.g., `chat_id`, `message_id`)
- Component props: camelCase
- File names: kebab-case for components, camelCase for utilities

### Error Handling
- API routes return appropriate HTTP status codes
- Client-side uses try/catch with console.error for debugging
- Database operations check for errors before proceeding

### Streaming Implementation
- Use ReadableStream for server-sent events
- Parse streaming chunks for reasoning vs. text content
- Update UI progressively during streaming, refresh from DB after completion

## Key Files & Examples

### Chat API (`src/app/api/chat/route.ts`)
```typescript
// Example: Storing AI response with reasoning
const content = `<reasoning-start>\n${reasoningText}\n<reasoning-end>\n\n<text-start>\n${text}\n<text-end>`;
await supabase.from("messages").insert({
    message_id: aiMessageId,
    content,
    role: "assistant",
    chat_id: chatId
});
```

### Response Parsing (`src/components/atom/response.tsx`)
```typescript
// Parse structured AI response format
function parseResponse(content: string): ParsedResponse {
    const reasoningStart = content.indexOf("<reasoning-start>");
    const reasoningEnd = content.indexOf("<reasoning-end>");
    // Extract reasoning and text sections
}
```

### Database Queries
```typescript
// Fetch chat messages ordered by creation time
const { data } = await supabase
    .from("messages")
    .select('message_id, content, role, created_at')
    .eq('chat_id', chatId)
    .order('created_at', { ascending: true });
```

## Integration Points

### Supabase Client Usage
- Server client: `src/util/supabase/server.ts` for API routes
- Client: `src/util/supabase/client.ts` for browser operations
- Always check `auth.getUser()` for authentication state

### AI SDK Configuration
- Uses `google("gemini-2.5-pro")` model
- Enables thinking with `thinkingConfig: { includeThoughts: true, thinkingBudget: 8192 }`
- Streams responses with `onFinish` callback for database storage

### Environment Variables
Required for local development:
- `SUPABASE_AUTH_EXTERNAL_GOOGLE_WEB_CLIENT_ID`
- `SUPABASE_AUTH_EXTERNAL_GOOGLE_CLIENT_SECRET`
- Google AI API key (configured in Supabase)

## Common Patterns

### Chat Creation Flow
1. Generate UUID for `chat_id`
2. Insert into `chats` table
3. Link user via `user_chats` table
4. Redirect to `/chat/${chat_id}`

### Message Storage
- User messages: Insert into `messages` and `user_msgs` (sender_type: "user")
- AI messages: Insert into `messages` and `user_msgs` (sender_type: "model", model_name: "gemini-2.5-pro")

### UI State Management
- Use React state for streaming content
- Refresh from database after streaming completes
- Handle temporary UI updates during loading states

- PS: Do not ever edit the migration files directly once created. Always create new migration files for schema changes.

## Debugging Tips
- Check Supabase Studio at http://localhost:54323 for database state
- Monitor streaming chunks in browser console
- Verify auth state with `supabase.auth.getUser()`
- Use database logs for migration issues</content>
<parameter name="filePath">c:\Users\Harshit Pandit\Desktop\pcm-chat\.github\copilot-instructions.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HarshitP147)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/HarshitP147)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
