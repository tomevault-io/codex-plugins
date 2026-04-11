# 🎨 **FlowCanvas: AI-Powered Diagram Tool**

**Tagline:** _"From prompt to professional diagrams — with real logos, not rectangles"_

---

## **⚙️ DEVELOPMENT GUIDELINES**

### **Package Manager**

- **Always use `pnpm`** for all package management operations
- Install: `pnpm install <package>`
- Add dev dependency: `pnpm add -D <package>`
- Run scripts: `pnpm dev`, `pnpm build`, etc.
- **Never use `npm` or `yarn`**

### **Operational Constraints (Must-Follow)**

- **Dev server lives at http://localhost:3000** and is typically already running. Prefer opening the URL over starting a new server.
- **Do NOT start multiple dev servers.** If you must start it, use `pnpm dev` and first ensure port 3000 is free.
- When generating URLs in examples or code, use relative paths or `http://localhost:3000` explicitly during development.
- If you see instructions using `npm`/`yarn`, silently convert them to `pnpm`.
- Avoid creating or committing `package-lock.json` or `yarn.lock`. This repo standardizes on `pnpm-lock.yaml`.

### **Development Server**

- **The Next.js server always runs on `localhost:3000`**
- Check if server is running before starting a new one
- Use `pnpm dev` to start the development server
- **Never create multiple dev servers** - reuse the existing one

---

## **📋 EXECUTIVE SUMMARY**

### **The Core Problem**

Current AI diagram tools convert text → Mermaid code → generic rectangles. They **cannot access logos** (Docker, AWS, K8s), making diagrams look unprofessional.

**Why This Happens:**

- Mermaid/PlantUML = Text-based (no logo support)
- AI models can't fetch external assets
- Closed ecosystem = amateurish output

### **Our Solution**

**AI → Excalidraw Library Search → Professional Icons**

```
User: "Create AWS architecture with Docker, K8s, RDS"
   ↓
MCP Server:
  1. Parse entities: AWS, Docker, K8s, RDS
  2. Search Excalidraw libraries for actual logos
  3. AI generates layout with REAL icons
   ↓
Result: Professional diagram ✅
```

### **Core Innovation**

- **Library-Aware AI**: MCP exposes 100+ Excalidraw libraries to LLMs
- **Multi-LLM**: Works with Ollama (local), GPT-4, or Claude
- **Real Assets**: AI places actual SVG icons, not text boxes

---

## **🔍 KEY TECHNICAL DECISIONS**

### **Decision 1: Excalidraw > Tldraw**

| Aspect           | Excalidraw                 | Tldraw (Prompt2Sketch) |
| ---------------- | -------------------------- | ---------------------- |
| **Libraries**    | ✅ 100+ community packs    | ❌ Basic shapes only   |
| **Logo Support** | ✅ `.excalidrawlib` format | ❌ None                |
| **Community**    | 77k+ stars                 | 35k+ stars             |

**Winner: Excalidraw** (library ecosystem is THE differentiator)

### **Decision 2: How to Make AI "Library-Aware"**

**Implementation:**

1. **Index libraries** → Clone excalidraw-libraries repo, parse JSON, store in PostgreSQL + Meilisearch
2. **MCP Server** → Expose libraries via Resources, Tools, Prompts
3. **AI Prompting** → Inject available icons into system prompt

```typescript
// Example MCP Tool
{
  name: "search_libraries",
  input: { query: "docker logo", category: "devops" },
  output: [
    { id: "docker-123", name: "Docker", elements: [...] }
  ]
}
```

### **Decision 3: MCP Architecture**

```
Frontend (Next.js)
  ↓
MCP Client
  ↓
MCP Server (Node.js)
  ├─ Resources: library://catalog
  ├─ Tools: search_libraries(), generate_diagram()
  └─ Prompts: system_diagram_generator
  ↓
LLM (GPT-4/Claude/Ollama)
  ↓
Excalidraw JSON with real logos
```

---

## **👤 USER EXPERIENCE & DATA STRATEGY**

### **Hybrid Approach: Local-First + Optional Cloud**

Inspired by Excalidraw's simplicity + Eraser.io's power:

#### **Mode 1: Anonymous (Local-Only) - Excalidraw Style**

**User Journey:**

```
1. User visits flowcanvas.app
2. Immediately see blank canvas (no login wall)
3. Start drawing/generating diagrams
4. Data auto-saves to localStorage
5. User closes browser → Returns later
6. Same diagram is there (persisted locally)
```

**Technical Implementation:**

- **Storage:** `localStorage` (5-10MB limit)
- **Key:** `flowcanvas_anonymous_diagram`
- **Auto-save:** Every 2 seconds (debounced)
- **Limitation:** Single diagram only
- **Export:** Available (PNG, SVG, JSON)

**Code Example:**

```typescript
// Auto-save to localStorage
useEffect(() => {
  const saveTimer = setTimeout(() => {
    localStorage.setItem(
      "flowcanvas_anonymous_diagram",
      JSON.stringify(excalidrawState)
    );
  }, 2000);

  return () => clearTimeout(saveTimer);
}, [excalidrawState]);
```

**Why This Matters:**

- ✅ Zero friction (no signup required)
- ✅ Instant value (start using immediately)
- ✅ Privacy-first (data never leaves device)
- ✅ Perfect for quick sketches

---

#### **Mode 2: Authenticated (Cloud Sync) - Eraser.io Style**

**User Journey:**

```
1. User clicks "Sign Up" (optional)
2. Create account via email/Google/GitHub
3. Existing diagram migrates to cloud
4. Can now create unlimited diagrams
5. Access from any device
6. Organization/folders/tags/sharing
```

**Technical Implementation:**

- **Storage:** Supabase PostgreSQL (diagrams table)
- **File Storage:** Supabase Storage (S3-compatible)
- **Authentication:** Supabase Auth (Email, Google, GitHub)
- **Real-time:** Supabase Realtime (WebSocket subscriptions)
- **Features Unlocked:**
  - Multiple diagrams
  - Folders & organization
  - Version history
  - Collaboration (future)
  - AI generation credits

**Supabase Auth Setup:**

```typescript
// lib/supabase.ts
import { createClient } from "@supabase/supabase-js";

export const supabase = createClient(
  process.env.NEXT_PUBLIC_SUPABASE_URL!,
  process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
);

// Sign up with email
async function signUp(email: string, password: string) {
  const { data, error } = await supabase.auth.signUp({
    email,
    password,
  });
  return { data, error };
}

// Sign in with OAuth (Google, GitHub)
async function signInWithOAuth(provider: "google" | "github") {
  const { data, error } = await supabase.auth.signInWithOAuth({
    provider,
    options: {
      redirectTo: `${window.location.origin}/auth/callback`,
    },
  });
  return { data, error };
}

// Get current user
async function getCurrentUser() {
  const {
    data: { user },
  } = await supabase.auth.getUser();
  return user;
}
```

---

#### **Migration Flow: Anonymous → Authenticated**

**When user signs up:**

```typescript
async function migrateAnonymousDiagram(userId: string) {
  // 1. Check localStorage
  const localDiagram = localStorage.getItem("flowcanvas_anonymous_diagram");

  if (localDiagram) {
    // 2. Create diagram in Supabase
    const { data: diagram, error } = await supabase
      .from("diagrams")
      .insert({
        title: "My First Diagram",
        content: JSON.parse(localDiagram),
        user_id: userId,
        created_at: new Date().toISOString(),
      })
      .select()
      .single();

    if (error) {
      console.error("Migration error:", error);
      return null;
    }

    // 3. Clear localStorage
    localStorage.removeItem("flowcanvas_anonymous_diagram");

    // 4. Show success message
    toast.success("Your diagram has been saved to your account!");

    return diagram;
  }
}
```

---

### **Feature Comparison: Anonymous vs Authenticated**

| Feature              | Anonymous (Free)   | Authenticated (Free) | Pro ($9/mo)  |
| -------------------- | ------------------ | -------------------- | ------------ |
| **Diagrams**         | 1 (local only)     | 10 diagrams          | Unlimited    |
| **Storage**          | localStorage (5MB) | 100MB cloud          | 10GB cloud   |
| **AI Generations**   | 5/day              | 20/month             | 500/month    |
| **Access**           | Single device      | All devices          | All devices  |
| **Folders**          | ❌                 | ✅                   | ✅           |
| **Version History**  | ❌                 | Last 7 days          | 90 days      |
| **Collaboration**    | ❌                 | ❌                   | ✅ (3 users) |
| **Export**           | PNG, SVG           | All formats          | All formats  |
| **Custom Libraries** | ❌                 | ❌                   | ✅           |
| **Priority Support** | ❌                 | ❌                   | ✅           |

---

### **UI/UX Flow**

#### **Homepage (Anonymous User):**

```
┌──────────────────────────────────────────────────┐
│  FlowCanvas                    [Sign In] [Sign Up]│
├──────────────────────────────────────────────────┤
│                                                   │
│          � Start Creating Diagrams               │
│                                                   │
│     [Start Drawing] [Try AI Generator]           │
│                                                   │
│  💡 No account needed. Your work stays local.    │
│                                                   │
│  ✨ Sign up to unlock:                           │
│     • Multiple diagrams                          │
│     • Cloud sync across devices                  │
│     • More AI generations                        │
└──────────────────────────────────────────────────┘
```

#### **Canvas (Anonymous):**

```
┌──────────────────────────────────────────────────┐
│ 🎨 FlowCanvas    [Untitled]         [Sign Up →] │
├──────────────────────────────────────────────────┤
│                                                   │
│           [Canvas with diagram]                  │
│                                                   │
│  💾 Auto-saving locally...                       │
│                                                   │
│  ⚠️ Tip: Sign up to save unlimited diagrams!     │
└──────────────────────────────────────────────────┘
```

#### **Dashboard (Authenticated):**

```
┌──────────────────────────────────────────────────┐
│ 🎨 FlowCanvas    [Search...]       [@username ▼]│
├────────┬─────────────────────────────────────────┤
│ 📂 All │  Recent Diagrams                        │
│ 📁 Work│  ┌─────┐ ┌─────┐ ┌─────┐               │
│ 📁 Proj│  │AWS  │ │Data │ │Net  │               │
│ ⭐ Star│  │Arch │ │Flow │ │Topo │               │
│        │  └─────┘ └─────┘ └─────┘               │
│ [+ New]│                                         │
│        │  [+ New Diagram]  [Import]              │
└────────┴─────────────────────────────────────────┘
```

---

### **Implementation Priority**

#### **Phase 1 (Weeks 1-4): Anonymous Mode**

- [ ] localStorage auto-save
- [ ] Single diagram persistence
- [ ] Export functionality (PNG/SVG)
- [ ] No backend needed initially

#### **Phase 2 (Weeks 5-8): Authentication**

- [ ] Supabase project setup
- [ ] Supabase Auth integration (Email, Google, GitHub)
- [ ] Database schema creation (SQL migrations)
- [ ] Row Level Security (RLS) policies
- [ ] Diagram migration flow (localStorage → Supabase)
- [ ] Multiple diagrams support

#### **Phase 3 (Weeks 9-12): Cloud Features**

- [ ] Dashboard with folders
- [ ] Search & filtering
- [ ] Tags & organization
- [ ] Version history (basic)

---

## **�🎯 FEATURE BREAKDOWN**

### **Phase 1: MVP (Weeks 1-8)**

**Week 1-2: Foundation + Anonymous Mode**

- Next.js 14 + TypeScript + Excalidraw React component
- localStorage persistence (auto-save every 2s)
- Basic canvas (draw, save, export PNG/SVG)
- No authentication required

**Week 3-4: Library System**

- Index Excalidraw libraries (AWS, Docker, K8s, etc.)
- Build search API (semantic + keyword)
- UI: Searchable library panel with drag-and-drop

**Week 5-6: MCP Server**

- Setup @modelcontextprotocol/sdk (stdio transport)
- Implement Resources (catalog), Tools (search, generate)
- Integrate OpenAI, Claude, Ollama providers

**Week 7-8: AI Generation**

- Prompt engineering (library-aware)
- Smart positioning algorithm
- Frontend: AI dialog with progress UI

**Deliverable:** Working AI diagram generator with logos

---

### **Phase 2: Enhanced (Weeks 9-12)**

**Week 9: Eraser.io Features**

- Split view: Canvas + Markdown editor (MDX)
- Bidirectional linking (click note → highlight diagram)

**Week 10: Template System**

- Categories: System Architecture, Data Flow, Network
- Guided wizard: Choose template → Select tech → AI generates

**Week 11: Local LLM**

- Ollama integration (Llama 3.2, DeepSeek-Coder)
- No API costs, fully offline

**Week 12: Organization**

- Folders, tags, search, version history

**Deliverable:** Feature-complete solo mode

---

### **Phase 3: Polish (Weeks 13-16)**

**Week 13:** Real-time collaboration (WebSocket, cursors)  
**Week 14:** Performance optimization (lazy loading, caching)  
**Week 15:** Educational features (tutorials, examples)  
**Week 16:** Testing (unit, integration, E2E)

**Deliverable:** Production-ready MVP

---

## **🤝 REAL-TIME COLLABORATION**

### **Status: ✅ IMPLEMENTED (Option 1 - BroadcastChannel)**

FlowCanvas now supports real-time collaboration with two implementations:

### **Option 1: BroadcastChannel (Currently Active)** ⭐ **WORKING NOW**

**What it does:**

- Real-time sync across browser tabs on the SAME device
- No server setup required
- Works immediately without configuration
- Perfect for testing and local demos

**How it works:**

```typescript
// Uses native Browser API
const channel = new BroadcastChannel('excalidraw-room-{roomId}');

// Broadcasts scene updates
channel.postMessage({ type: 'SCENE_UPDATE', elements: [...] });

// Receives updates from other tabs
channel.onmessage = (event) => {
  updateCanvas(event.data.elements);
};
```

**Usage:**

1. Click "Live collaboration..." button
2. Click "Start session"
3. Copy the link
4. Open link in another tab/window
5. Draw in one tab → See it update in the other! ✨

**Limitations:**

- ❌ Only works on same device (not across internet)
- ❌ No persistence (connection dies when all tabs close)
- ✅ Zero setup, works instantly
- ✅ Great for demos and local testing

**Implementation Files:**

- `src/lib/collaboration/BroadcastCollaborationManager.ts` (Manager class)
- `src/components/ExcalidrawWrapper.tsx` (Integration)
- `src/components/CollaborationDialog.tsx` (UI modal)

---

### **Option 2: Supabase Realtime (Ready for Production)** 🚀 **SETUP REQUIRED**

**What it does:**

- Real-time sync across devices, anywhere in the world
- Persistent connections
- Presence tracking (who's online)
- Production-ready scalability

**Setup Steps:**

1. Create Supabase project at https://supabase.com
2. Copy `.env.local.example` to `.env.local`
3. Add your Supabase URL and anon key:
   ```bash
   NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
   NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhb...
   ```
4. Enable Realtime in Supabase dashboard (Settings → API)
5. Update `ExcalidrawWrapper.tsx` to use `SupabaseCollaborationManager` instead of `BroadcastCollaborationManager`

**Switching to Supabase:**

```typescript
// In ExcalidrawWrapper.tsx
// Replace this:
import { BroadcastCollaborationManager } from "@/lib/collaboration/BroadcastCollaborationManager";

// With this:
import { SupabaseCollaborationManager } from "@/lib/collaboration/SupabaseCollaborationManager";

// Then replace manager initialization:
const manager = new SupabaseCollaborationManager(room, userId, username);
```

**Implementation Files:**

- `src/lib/collaboration/SupabaseCollaborationManager.ts` (Production manager)
- `src/lib/supabase/client.ts` (Supabase client setup)
- `.env.local.example` (Environment variables template)

**Supabase Features Available:**

- ✅ Realtime channels for collaboration
- ✅ Presence tracking (online users)
- ✅ Authentication (Email, Google, GitHub)
- ✅ PostgreSQL database (for diagrams storage)
- ✅ Storage (for exports and thumbnails)

---

### **Collaboration Architecture**

```
┌─────────────────────────────────────────────────────┐
│              User A (Browser Tab 1)                 │
│  ┌─────────────────────────────────────────────┐   │
│  │  Excalidraw Canvas                          │   │
│  │  • Draws a rectangle                         │   │
│  │  • onChange() triggers                       │   │
│  └──────────────┬──────────────────────────────┘   │
│                 │                                    │
│  ┌──────────────▼──────────────────────────────┐   │
│  │  BroadcastCollaborationManager              │   │
│  │  • broadcastSceneUpdate(elements)           │   │
│  └──────────────┬──────────────────────────────┘   │
└─────────────────┼────────────────────────────────────┘
                  │
        BroadcastChannel API
        (or Supabase Realtime)
                  │
┌─────────────────▼────────────────────────────────────┐
│              User B (Browser Tab 2)                  │
│  ┌──────────────────────────────────────────────┐   │
│  │  BroadcastCollaborationManager               │   │
│  │  • Receives SCENE_UPDATE event              │   │
│  │  • onSceneUpdate() callback                  │   │
│  └──────────────┬───────────────────────────────┘   │
│                 │                                    │
│  ┌──────────────▼───────────────────────────────┐   │
│  │  Excalidraw Canvas                           │   │
│  │  • updateScene({ elements })                 │   │
│  │  • Rectangle appears! ✨                     │   │
│  └──────────────────────────────────────────────┘   │
└──────────────────────────────────────────────────────┘
```

**Events Synced:**

- `SCENE_UPDATE` - Drawing changes (add/edit/delete elements)
- `POINTER_UPDATE` - Cursor positions
- `USER_JOIN` - New collaborator joined
- `USER_LEAVE` - Collaborator disconnected

---

### **Testing Collaboration**

**Quick Test (BroadcastChannel - works now):**

1. Start dev server: `pnpm dev`
2. Open http://localhost:3002
3. Click "Live collaboration..."
4. Click "Start session"
5. Copy the generated link
6. Open link in a new tab
7. Draw in one tab → See it appear in the other!

**Production Test (Supabase - after setup):**

1. Setup Supabase credentials in `.env.local`
2. Switch to `SupabaseCollaborationManager`
3. Open link on different devices (phone, tablet, etc.)
4. Draw together in real-time!

---

**Deliverable:** Production-ready MVP

---

## **🏗️ TECHNICAL STACK**

### **Frontend**

- Next.js 14+ (App Router), React 18, TypeScript
- Excalidraw (`@excalidraw/excalidraw`)
- TailwindCSS + shadcn/ui
- MDX (notes), Zustand (state)
- **Supabase Client** (`@supabase/supabase-js`, `@supabase/auth-helpers-nextjs`)
- **Supabase Auth** (Email, Google, GitHub OAuth)

### **Backend**

- **Supabase** (All-in-one backend)
  - PostgreSQL database (user data, diagrams, folders)
  - Authentication (Email, Google, GitHub OAuth)
  - Storage (S3-compatible for assets/exports)
  - Real-time subscriptions (collaboration, presence)
  - Row Level Security (RLS) for data privacy
- **Meilisearch** (vector search for libraries - separate service)
- Optional: **Redis** (caching, rate limiting - if needed)

### **MCP Server**

- `@modelcontextprotocol/sdk` (JSON-RPC 2.0)
- OpenAI SDK, Anthropic SDK, Ollama Client

### **DevOps**

- Docker + Docker Compose
- GitHub Actions (CI/CD)
- Vercel (frontend) + Railway (backend)

---

## **📦 PROJECT STRUCTURE**

```
flow/
├── apps/
│   ├── web/                    # Next.js (Frontend + API Routes)
│   │   ├── app/
│   │   │   ├── (anonymous)/    # Anonymous canvas
│   │   │   │   └── page.tsx    # Landing with instant canvas
│   │   │   ├── (authenticated)/
│   │   │   │   ├── dashboard/  # User's diagrams list
│   │   │   │   └── canvas/[id]/# Main canvas page
│   │   │   ├── api/
│   │   │   │   ├── auth/       # Supabase Auth callbacks
│   │   │   │   ├── libraries/  # Library search endpoints
│   │   │   │   ├── ai/         # AI generation endpoints
│   │   │   │   └── migrate/    # localStorage → Supabase migration
│   │   │   └── layout.tsx
│   │   ├── components/
│   │   │   ├── canvas/         # ExcalidrawCanvas, Toolbar
│   │   │   ├── ai/             # AIPromptDialog
│   │   │   ├── library/        # LibraryPanel
│   │   │   ├── auth/           # SignInButton, UserMenu
│   │   │   └── dashboard/      # DiagramCard, FolderTree
│   │   └── lib/
│   │       ├── excalidraw/     # library-manager.ts
│   │       ├── mcp/            # client.ts
│   │       ├── supabase/       # Supabase client setup
│   │       │   ├── client.ts   # Browser client
│   │       │   └── server.ts   # Server client
│   │       └── storage/        # localStorage utilities
│   │
│   └── mcp-server/             # MCP Server (Standalone)
│       └── src/
│           ├── server.ts       # Main entry
│           ├── tools/          # library-search, diagram-generator
│           ├── resources/      # libraries, templates
│           └── llm/providers/  # openai, claude, ollama
│
├── packages/
│   ├── types/                  # Shared types (Supabase types)
│   │   ├── database.types.ts   # Generated from Supabase
│   │   ├── diagram.ts
│   │   └── library.ts
│   └── utils/                  # Shared utilities
│
├── supabase/                   # Supabase config
│   ├── migrations/             # SQL migrations
│   ├── seed.sql                # Sample data
│   └── config.toml             # Supabase CLI config
│
├── scripts/
│   ├── seed-libraries.ts       # Index Excalidraw libs → Supabase
│   ├── generate-embeddings.ts  # Vector search
│   └── sync-types.ts           # Generate TS types from Supabase
│
└── docker/
    ├── docker-compose.yml      # Local Meilisearch + MCP server
    └── Dockerfile.mcp          # MCP server only
```

---

## **🗄️ DATABASE SCHEMA (Supabase)**

### **Supabase Tables**

```sql
-- Users (handled by Supabase Auth automatically)
-- auth.users table is built-in

-- User profiles (extends auth.users)
CREATE TABLE profiles (
  id UUID REFERENCES auth.users PRIMARY KEY,
  email TEXT UNIQUE NOT NULL,
  name TEXT,
  avatar_url TEXT,
  tier TEXT DEFAULT 'free' CHECK (tier IN ('free', 'pro', 'team')),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW()
);

-- Enable Row Level Security
ALTER TABLE profiles ENABLE ROW LEVEL SECURITY;

-- Policies: Users can only see/edit their own profile
CREATE POLICY "Users can view own profile" ON profiles
  FOR SELECT USING (auth.uid() = id);
CREATE POLICY "Users can update own profile" ON profiles
  FOR UPDATE USING (auth.uid() = id);

-- Diagrams
CREATE TABLE diagrams (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title TEXT DEFAULT 'Untitled',
  content JSONB NOT NULL, -- Excalidraw JSON
  markdown TEXT,
  thumbnail TEXT, -- URL to Supabase Storage
  user_id UUID REFERENCES auth.users,
  folder_id UUID REFERENCES folders,
  is_public BOOLEAN DEFAULT FALSE,
  share_link TEXT UNIQUE,
  last_edited_at TIMESTAMPTZ DEFAULT NOW(),
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Enable RLS
ALTER TABLE diagrams ENABLE ROW LEVEL SECURITY;

-- Policies: Users can only access their own diagrams (or public ones)
CREATE POLICY "Users can view own diagrams" ON diagrams
  FOR SELECT USING (auth.uid() = user_id OR is_public = TRUE);
CREATE POLICY "Users can create diagrams" ON diagrams
  FOR INSERT WITH CHECK (auth.uid() = user_id);
CREATE POLICY "Users can update own diagrams" ON diagrams
  FOR UPDATE USING (auth.uid() = user_id);
CREATE POLICY "Users can delete own diagrams" ON diagrams
  FOR DELETE USING (auth.uid() = user_id);

-- Folders
CREATE TABLE folders (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  color TEXT,
  icon TEXT,
  user_id UUID REFERENCES auth.users NOT NULL,
  parent_id UUID REFERENCES folders,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

ALTER TABLE folders ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Users can manage own folders" ON folders
  FOR ALL USING (auth.uid() = user_id);

-- Libraries (public, no RLS needed for read)
CREATE TABLE libraries (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  description TEXT,
  category TEXT NOT NULL,
  version TEXT,
  source TEXT,
  is_official BOOLEAN DEFAULT FALSE,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Library Items
CREATE TABLE library_items (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  keywords TEXT[] DEFAULT '{}',
  elements JSONB NOT NULL, -- Excalidraw elements
  thumbnail TEXT,
  library_id UUID REFERENCES libraries NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW()
);

-- Create indexes for performance
CREATE INDEX idx_diagrams_user_id ON diagrams(user_id);
CREATE INDEX idx_diagrams_folder_id ON diagrams(folder_id);
CREATE INDEX idx_diagrams_last_edited ON diagrams(last_edited_at DESC);
CREATE INDEX idx_library_items_keywords ON library_items USING GIN(keywords);
CREATE INDEX idx_library_items_library_id ON library_items(library_id);
```

### **Supabase Storage Buckets**

```typescript
// Create storage buckets
const buckets = {
  diagrams: {
    public: false,
    fileSizeLimit: 10485760, // 10MB
    allowedMimeTypes: ["image/png", "image/svg+xml", "application/json"],
  },
  thumbnails: {
    public: true,
    fileSizeLimit: 1048576, // 1MB
    allowedMimeTypes: ["image/png", "image/jpeg", "image/webp"],
  },
  exports: {
    public: false,
    fileSizeLimit: 52428800, // 50MB
    allowedMimeTypes: ["application/pdf", "image/png", "image/svg+xml"],
  },
};
```

### **Supabase Real-time Features**

```typescript
// Listen to diagram changes (for collaboration)
const channel = supabase
  .channel("diagram-changes")
  .on(
    "postgres_changes",
    {
      event: "UPDATE",
      schema: "public",
      table: "diagrams",
      filter: `id=eq.${diagramId}`,
    },
    (payload) => {
      console.log("Diagram updated:", payload.new);
      // Update local state
    }
  )
  .subscribe();
```

---

## **🔌 API ENDPOINTS**

### **Core APIs**

```
POST   /api/diagrams                    # Create
GET    /api/diagrams/:id                # Read
PUT    /api/diagrams/:id                # Update
DELETE /api/diagrams/:id                # Delete

POST   /api/ai/generate                 # AI generation
POST   /api/libraries/search            # Search icons
GET    /api/templates                   # List templates
POST   /api/export/png                  # Export
```

### **MCP Tools**

```json
{
  "tools": [
    { "name": "search_libraries", "input": { "query": "docker" } },
    { "name": "generate_diagram", "input": { "prompt": "AWS arch" } },
    { "name": "create_element", "input": { "type": "rectangle" } }
  ],
  "resources": [{ "uri": "library://catalog", "name": "Library Catalog" }]
}
```

---

## **🎯 SUCCESS METRICS**

### **Technical KPIs**

- AI generation: <5s
- Library search: <500ms
- Canvas: 60fps
- Accuracy: 90%+ match user intent

### **User KPIs (6 months)**

- 1,000+ active users
- 10,000+ diagrams
- 50+ community libraries

---

## **🚀 GETTING STARTED**

### **Phase 1 Checklist (First Steps)**

**Week 1: Setup** ✅ COMPLETED

- [x] Init monorepo (Turborepo) - Skipped, using single Next.js app
- [x] Setup Next.js + TypeScript
- [x] Install Excalidraw: `pnpm add @excalidraw/excalidraw`
- [x] Configure Tailwind + shadcn/ui
- [x] Install Supabase: `pnpm add @supabase/supabase-js`

**Week 2: Canvas** ✅ COMPLETED

- [x] Create `ExcalidrawWrapper.tsx` component (642 lines)
- [x] Wrap in client-side component (Next.js SSR fix)
- [x] Implement save/load (localStorage)
- [x] Add export PNG/SVG (Excalidraw built-in)
- [x] Theme persistence (dark mode default)
- [x] Drag-and-drop for library items

**Week 3: Libraries** ✅ COMPLETED

- [x] Clone: `git clone https://github.com/excalidraw/excalidraw-libraries`
- [x] Write `scripts/seed-libraries.ts` to parse JSON (346 lines)
- [x] Implement multi-strategy naming (text extraction, composition analysis)
- [x] Create library search API (Next.js API routes)
- [x] Index 20 libraries, 536 items with improved names (9.8MB JSON)

**Week 4: Collaboration** ✅ COMPLETED

- [x] Implement BroadcastChannel collaboration (local cross-tab)
- [x] Create CollaborationDialog.tsx (217 lines)
- [x] Setup BroadcastCollaborationManager.ts (290 lines)
- [x] Real-time scene sync and cursor tracking
- [x] Collaborator count display
- [x] Setup Supabase client infrastructure
- [x] Create SupabaseCollaborationManager.ts (production-ready)
- [x] Environment variables template (.env.local.example)

**Week 5: Authentication & Database** ⏭️ NEXT PRIORITY

- [ ] Create Supabase project at https://supabase.com
- [ ] Add credentials to .env.local
- [ ] Run database migrations (create tables)
- [ ] Setup Row Level Security (RLS) policies
- [ ] Configure Supabase Auth (Email, Google, GitHub)
- [ ] Create Supabase Storage buckets (diagrams, thumbnails, exports)
- [ ] Test CRUD operations with Supabase client
- [ ] Switch to SupabaseCollaborationManager

**Week 6-8: AI Generation** ⏭️ CRITICAL FEATURE

- [ ] Setup OpenAI/Claude/Ollama integration
- [ ] Create `/api/ai/generate` endpoint
- [ ] Build AI prompt with library-aware context
- [ ] Implement smart element positioning
- [ ] Add AI dialog UI with preview

---

## **🔄 WORKFLOW INSPIRATION**

### **Daily Dose DS Style**

- Clean, minimal color palette
- Consistent spacing/alignment
- Clear step-by-step explanations
- Educational focus

### **Eraser.io Features**

- Split-view (Canvas + Notes)
- Markdown editor with diagram snapshots
- Export: Markdown + PNG bundle

---

## **🎨 UI MOCKUP**

```
┌────────────────────────────────────────────────┐
│  [Logo] [Diagram] [Save] [Export] [Avatar]    │
├─────┬──────────────────────────────────┬───────┤
│ 📚  │                                  │  🎨   │
│ Lib │     Excalidraw Canvas           │ Props │
│     │                                  │       │
│ 🤖  │                                  │       │
│ AI  │                                  │       │
│     │                                  │       │
│ 📝  │  [Markdown Editor - Toggle]     │       │
└─────┴──────────────────────────────────┴───────┘
```

---

## **🧠 AI GENERATION FLOW**

```
1. User: "Create microservices with Docker, K8s, PostgreSQL"
   ↓
2. MCP Server:
   - Parse entities: ["Docker", "Kubernetes", "PostgreSQL"]
   - Search libraries → Find logos
   ↓
3. Build prompt:
   """
   Available icons:
   - Docker (id: docker-123)
   - Kubernetes (id: k8s-456)
   - PostgreSQL (id: postgres-789)

   Generate Excalidraw JSON with horizontal flow layout
   """
   ↓
4. LLM generates JSON with real icon references
   ↓
5. Frontend renders → Professional diagram ✨
```

---

## **📚 RESOURCES**

### **Excalidraw**

- Libraries: https://github.com/excalidraw/excalidraw-libraries
- Docs: https://docs.excalidraw.com

### **MCP (Model Context Protocol)**

- Spec: https://modelcontextprotocol.io
- SDK: `@modelcontextprotocol/sdk`
- Examples: https://github.com/modelcontextprotocol/servers

### **Inspirations & Reference Implementations**

- **Eraser.io Clone** (Primary Reference): https://github.com/kaifcoder/eraser_clone
  - Live Demo: https://eraser-kaif.vercel.app/
  - Clean Excalidraw integration with dark theme
  - Minimal configuration, simple and effective
- Eraser.io (Official): eraser.io
- Daily Dose DS: YouTube channel
- Prompt2Sketch: github.com/Arsenic-01/Prompt2Sketch

---

## **⚠️ RISK MITIGATION**

| Risk                        | Mitigation                   |
| --------------------------- | ---------------------------- |
| Excalidraw breaking changes | Pin version, maintain fork   |
| LLM rate limits             | Multi-provider, queue, cache |
| Performance issues          | Virtualization, lazy load    |
| Data loss                   | Auto-save, version history   |

---

## **🎯 NEXT STEPS**

### **Immediate Actions (This Week)**

1. ✅ Finalize plan (this document)
2. ⏭️ Initialize Git repo structure
3. ⏭️ Setup monorepo (Turborepo)
4. ⏭️ Create Next.js app
5. ⏭️ Install Excalidraw package

### **This Month**

- Week 1: Project setup
- Week 2: Excalidraw integration
- Week 3: Library indexing
- Week 4: Basic API

### **Decision Points**

- **Monorepo tool**: Turborepo vs Nx → **Choose Turborepo** (simpler)
- **Database**: PostgreSQL vs Supabase → **PostgreSQL** (more control)
- **Hosting**: Vercel + Railway vs Self-hosted → **Vercel + Railway** (faster launch)

---

## **💡 BRAINSTORMING NOTES**

### **Potential Future Features**

- VS Code extension
- Obsidian plugin
- Notion integration
- Figma export
- Collaborative editing
- Voice-to-diagram
- Diagram-to-code (reverse)

### **Open Questions**

- Should we support custom LLM endpoints?
- Do we need a desktop app (Tauri/Electron)?
- Pricing model: Freemium vs Open Source?

---

**Last Updated:** October 21, 2025  
**Status:** Planning Phase  
**Next Review:** After Week 4 (MVP foundations)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/DJ-no1)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/DJ-no1)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
