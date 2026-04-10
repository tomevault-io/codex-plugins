# BuildMate Studio - AI Coding Agent Instructions

## Platform Vision & Mission
BuildMate Studio is a spec-driven, collaborative platform where anyone can describe an idea and receive a production-ready product across web and native mobile, with governance and auditability suitable for teams and enterprises.

**Mission**: Make software creation as simple as describing what you want, then deliver a secure, observable, and shippable product that matches that description.

**Core Goals**:
- Turn natural-language ideas into a single source of truth (the Spec) that guides everything produced
- Deliver complete, usable applications—web and native mobile—that feel coherent, polished, and ready for real users
- Support multiplayer creation so teams can co-create, review, and approve work in one place
- Ensure outcomes meet business needs: clear user journeys, sensible data models, accessible interfaces, and measurable success
- Provide smooth paths to launch, update, and roll back across environments without stress

## Architecture Overview
BuildMate Studio is a full-stack AI-powered app builder with React frontend, Express backend, and PostgreSQL database via Drizzle ORM. The app supports real-time collaborative editing, PWA functionality, and enterprise features.

**Service Boundaries:**
- `client/src/` - React SPA with TypeScript, Tailwind CSS, shadcn/ui components
- `server/` - Express.js API server with session auth and WebSocket support
- `shared/` - Common TypeScript types and database schema
- `migrations/` - Drizzle database migrations

**Data Flow:**
- Frontend uses TanStack Query for API calls with CSRF protection
- Backend uses session-based authentication shared between HTTP/WebSocket
- Real-time collaboration via Yjs over WebSocket connections
- Database operations through Drizzle ORM with PostgreSQL

## End-State Outcomes
- **Living Specification**: Captures purpose, audience, user journeys, data, roles, and success criteria in plain language
- **Working Applications**: Web apps that reflect specification intents, brand tone, accessibility expectations
- **Native Mobile Apps**: iOS and Android apps that mirror product vision, handle offline gracefully, respect platform conventions
- **Clear Documentation**: Human-readable docs explaining what was built, why it matters, and how to operate
- **Governance**: Visible decision trails, privacy posture, and licensing clarity for enterprise adoption

## Key Product Templates
- **Subscriptions/Memberships**: Support plans, trials, and renewals
- **Ticketing/Bookings**: Capacity management and check-in flows
- **CRM-lite**: Contacts, pipelines, and simple follow-ups

## Development Principles
- **Clarity over cleverness**: Prefer understandable outcomes to obscure tricks
- **Safety over speed**: When user trust, privacy, or compliance is at stake
- **Portability**: Avoid decisions that trap the product in one vendor or tool
- **Accessibility by default**: Plain language, keyboard and screen-reader friendly
- **Multi-platform consistency**: Same description defines web and native mobile experiences

## Critical Developer Workflows

### Development Server
```bash
npm run dev  # Runs Express server with Vite middleware for HMR
```

### Building for Production
```bash
npm run build      # Builds client to dist/public, bundles server to dist/index.js
npm run build:dev  # Development build without minification
npm start          # Runs production server from dist/
```

### Database Operations
```bash
npm run db:push  # Applies schema changes to database
```

### Testing
```bash
npm run check  # TypeScript type checking
```

## Project-Specific Conventions

### Authentication & Security
- **Session-based auth**: Use `req.session.user` for authentication checks
- **CSRF protection**: Apply `csrfProtection` middleware to all state-changing POST/PUT/DELETE routes
- **Raw body parsing**: Webhooks at `/api/webhooks/*` get raw bytes before JSON parsing
- **Session sharing**: WebSocket upgrades run session middleware for auth

### API Patterns
- **CSRF tokens**: Fetch from `/api/csrf-token`, include in state-changing requests
- **Error handling**: Return `{ message: string }` for client errors
- **Health checks**: `/api/health` and `/api/session` endpoints available

### Frontend Patterns
- **Path aliases**: `@/` for `client/src/`, `@shared/` for `shared/`, `@assets/` for `attached_assets/`
- **Routing**: Use wouter library instead of React Router
- **Components**: shadcn/ui components in `client/src/components/ui/`
- **State management**: TanStack Query for server state, local state as needed

### Database Patterns
- **Schema location**: `shared/schema.ts` defines all tables and relations
- **Migrations**: Auto-generated in `migrations/` directory
- **Relations**: Use Drizzle's `relations()` helper for foreign keys
- **Zod schemas**: Auto-generated from Drizzle schema for validation

## Integration Points

### External Services
- **Supabase**: Planned migration for auth/database (currently using direct PostgreSQL)
- **OpenAI**: AI code generation features
- **GitHub**: OAuth integration and repository operations
- **Replit**: Deployment platform with autoscale/static/reserved VM options

### Real-time Features
- **WebSocket**: Session-authenticated connections for real-time updates
- **Yjs**: CRDT-based collaborative editing with Monaco editor integration
- **Service Worker**: PWA functionality with offline support

### Development Tools
- **Vite**: Development server with HMR, special Codespaces config for tunneling
- **Drizzle Kit**: Database migrations and schema management
- **Replit plugins**: Cartographer and dev banner for development
- **Monaco Editor**: Code editing with Yjs real-time collaboration

## Governance & Quality Standards
- **Every change connected to intent**: Easy to review in context of stated goals
- **User data treated respectfully**: Documented in plain language
- **Dependencies visible**: Third-party licenses appropriate for intended use
- **Success metrics**: Short time to first release, stakeholder recognition, smooth releases

## Common Patterns & Examples

### Adding a new API route:
```typescript
// server/routes.ts - Add after CSRF protection setup
app.post("/api/new-endpoint", csrfProtection, (req, res) => {
  if (!(req.session as any)?.user) {
    return res.status(401).json({ error: "Unauthorized" });
  }
  // Implementation
});
```

### Adding a database table:
```typescript
// shared/schema.ts
export const newTable = pgTable("new_table", {
  id: varchar("id").primaryKey().default(sql`gen_random_uuid()`),
  // fields...
});
```

### Frontend API call:
```typescript
// Use TanStack Query with CSRF token
const { data } = useQuery({
  queryKey: ["/api/data"],
  queryFn: async () => {
    const token = await getCSRFToken();
    return apiRequest("/api/data", {
      method: "POST",
      headers: { "X-CSRF-Token": token },
      body: JSON.stringify(payload)
    });
  }
});
```

### Component structure:
```tsx
// client/src/components/NewComponent.tsx
import { Button } from "@/components/ui/button";

export function NewComponent() {
  return <Button>Click me</Button>;
}
```</content>
<parameter name="filePath">/workspaces/fertile-ground-base/.github/copilot-instructions.md

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/themateplatform)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/themateplatform)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
