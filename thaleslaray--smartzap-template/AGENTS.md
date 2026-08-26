# SmartZap - Copilot Instructions

## Project Overview
WhatsApp marketing automation SaaS built with Next.js 16, React 19, Turso (SQLite), Upstash Redis/QStash. Integrates Meta WhatsApp Cloud API (v24.0) for templated messages and Gemini AI for content generation.

## Architecture

### Frontend: Page → Hook → Service → API
```
app/(dashboard)/campaigns/page.tsx  # Thin page: wires hook to view
    ↓
hooks/useCampaigns.ts               # Controller hook: React Query + UI state
    ↓
services/campaignService.turso.ts   # API calls to Next.js routes
    ↓
app/api/campaigns/route.ts          # API Route → Turso DB
```

### Backend: Serverless + Queues
```
API Routes (Next.js)  →  QStash Workflow  →  Meta WhatsApp API
        ↓                     ↓
    Turso DB             Redis Cache
```

## Key Conventions

### Component Pattern (View + Controller separation)
```tsx
// components/features/campaigns/CampaignListView.tsx - PURE presentational
interface CampaignListViewProps {
  campaigns: Campaign[];
  onDelete: (id: string) => void;  // Events as callbacks
  onRowClick: (id: string) => void;
}

// hooks/useCampaigns.ts - Controller hook pattern
export const useCampaignsController = () => {
  const { data } = useCampaignsQuery();           // React Query
  const [filter, setFilter] = useState('All');    // UI state
  const filteredCampaigns = useMemo(() => ...);   // Derived state
  return { campaigns, filter, setFilter, onDelete };
};

// app/(dashboard)/campaigns/page.tsx - Thin page (just wires hook to view)
```

### Database Layer (`lib/turso.ts`)
```typescript
turso.execute('SELECT * FROM campaigns')  // Direct SQL
campaignDb.getAll()                        // Abstracted CRUD
campaignDb.create({ name, templateName })
```

### Error Handling (`lib/whatsapp-errors.ts`)
```typescript
// 44 WhatsApp error codes mapped
mapWhatsAppError(131042)  // → { type: 'payment', message: '...', action: '...' }
isCriticalError(code)     // Payment, auth errors
isOptOutError(code)       // User blocked business
```

### Phone Number Handling (`lib/phone-formatter.ts`)
```typescript
normalizePhoneNumber('+5511999999999')  // E.164 format required by WhatsApp
validatePhoneNumber(phone)               // Uses libphonenumber-js
```

## Development Commands
```bash
npm run dev          # Next.js dev server (Turbopack)
npm run build        # Production build
npm run lint         # ESLint check
```

## Meta WhatsApp API (v24.0)

### Template Payload Structure
```json
{
  "messaging_product": "whatsapp",
  "to": "+5511999999999",
  "type": "template",
  "template": {
    "name": "template_name",
    "language": { "code": "pt_BR" },
    "components": [
      { "type": "header", "parameters": [{ "type": "image", "image": { "id": "..." } }] },
      { "type": "body", "parameters": [{ "type": "text", "text": "{{1}} value" }] }
    ]
  }
}
```

### Rate Limits
- **Cloud API**: Up to 1000 msgs/sec
- **Pair limit**: 1 msg/6 sec to same user (error 131056)
- **Retry**: Exponential backoff per Meta recommendation

## Environment Variables
```env
# Database (Required)
TURSO_DATABASE_URL=     # libsql://...
TURSO_AUTH_TOKEN=       # Turso auth token

# Redis (Required)
UPSTASH_REDIS_REST_URL=
UPSTASH_REDIS_REST_TOKEN=

# Queues (Required)
QSTASH_TOKEN=
QSTASH_CURRENT_SIGNING_KEY=
QSTASH_NEXT_SIGNING_KEY=

# AI (Optional)
GEMINI_API_KEY=

# Auth (Required)
AUTH_SECRET=            # 32+ character secret
```

## Data Storage Architecture

### Turso (SQLite) - Persistent Data
- Campaigns, contacts, flows, templates
- User sessions and auth
- Flow executions and node executions
- Dashboard stats

### Redis (Upstash) - Credentials & Cache
- **WhatsApp credentials**: `settings:whatsapp:credentials`
  ```typescript
  // Stored as JSON with:
  { phoneNumberId, businessAccountId, accessToken, displayPhoneNumber, verifiedName }
  ```
- **Conversation state**: `conversation:{phoneNumberId}:{contactPhone}`
- **Rate limiting**: `ratelimit:*`
- **Session cache**: Temporary session data

### Important: Credentials Location
WhatsApp API credentials (accessToken, phoneNumberId) are stored in **Redis**, NOT Turso.
```typescript
// To fetch credentials:
import { redis } from '@/lib/redis'
const credentials = await redis.get('settings:whatsapp:credentials')

// Fallback to env vars:
process.env.WHATSAPP_PHONE_ID || process.env.WHATSAPP_PHONE_NUMBER_ID
process.env.WHATSAPP_TOKEN || process.env.WHATSAPP_ACCESS_TOKEN
```

## Styling (Tailwind CSS v4)
- Primary: `primary-400`, `primary-500`, `primary-600` (emerald/green)
- Backgrounds: `zinc-800`, `zinc-900`, `zinc-950`
- Glass effect: `glass-panel` utility class
- Icons: lucide-react exclusively
- All UI text: Portuguese (pt-BR)

## Types (`types.ts`)
```typescript
CampaignStatus: DRAFT | SCHEDULED | SENDING | COMPLETED | PAUSED | FAILED
TemplateCategory: 'MARKETING' | 'UTILIDADE' | 'AUTENTICACAO'
ContactStatus: OPT_IN | OPT_OUT | UNKNOWN
```

## File Structure Quick Reference
```
app/
├── (dashboard)/          # Authenticated routes
│   ├── page.tsx          # Dashboard
│   ├── campaigns/        # Campaign management
│   ├── contacts/         # Contact management
│   ├── templates/        # Template viewer
│   └── settings/         # App settings
├── api/                  # API Routes
│   ├── campaigns/        # Campaign CRUD
│   ├── contacts/         # Contact CRUD
│   ├── webhook/          # Meta webhook handler
│   └── account/alerts/   # Account alerts API

components/
├── features/             # Feature-specific views
└── ui/                   # Reusable UI components

hooks/                    # React Query hooks (controller pattern)
services/                 # API client services
lib/
├── turso.ts              # Database client
├── turso-db.ts           # Database operations
├── redis.ts              # Cache client
├── whatsapp-errors.ts    # Error code mapping
└── phone-formatter.ts    # Phone validation

types.ts                  # TypeScript types
```

## Known Issues

### Delete/Duplicate Flash-back
Due to Vercel edge cache (10s TTL), deleted items may momentarily reappear on page refresh. The item is deleted in DB immediately but cached response may still contain it.

### Payment Alert (Error 131042)
When Meta account has payment issues, banner shows automatically. Auto-dismisses when delivery succeeds after payment is fixed.

## Flow Engine (Workflow Builder)

### Architecture
```
components/features/workflow-v2/     # Visual workflow editor (ReactFlow)
lib/flow-engine/                     # Execution engine
├── executor.ts                      # Core step execution
├── state.ts                         # Conversation state management
├── sender.ts                        # WhatsApp message sending
├── variables.ts                     # Variable interpolation
└── nodes/                           # Node-specific handlers
```

### Node Types
```typescript
type NodeType = 
  | 'start'       // Entry point (required)
  | 'message'     // Send text message
  | 'template'    // Send WhatsApp template
  | 'menu'        // Interactive buttons/list
  | 'input'       // Collect user input
  | 'condition'   // Branching logic
  | 'delay'       // Wait before next step
  | 'ai_agent'    // AI-powered response
  | 'handoff'     // Transfer to human
  | 'end'         // End conversation (optional)
```

### Test Flow API
```typescript
// POST /api/flows/test
// Executes flow for configured test contact
// Credentials fetched from Redis: settings:whatsapp:credentials
// Test contact from localStorage: smartzap_settings.testContact
```

### Execution Modes
- **chatbot**: Real-time response to incoming messages
- **campaign**: Batch send to contact list via QStash

---
> Source: [thaleslaray/smartzap_template](https://github.com/thaleslaray/smartzap_template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-26 -->
