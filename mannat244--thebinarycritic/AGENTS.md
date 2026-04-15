# The Binary Critic (TBC) - AI Coding Instructions

## Project Architecture
- **Framework**: Next.js 16 (App Router) with React 19.
- **Database**: MongoDB with Mongoose (`lib/db.js` handles connection reuse).
- **Authentication**: NextAuth.js v4 (`app/api/auth/[...nextauth]/route.js`) supporting Google & Credentials.
- **Styling**: Tailwind CSS v4, Radix UI primitives, and `shadcn/ui`-like components in `components/ui/`.
- **State/Cache**: Client-side caching via IndexedDB (`lib/idb.js`) and `browserCache.js`.
- **Vector Search**: `User` model references `embeddingId`, implying Pinecone or similar integration.

## Core Patterns & Conventions

### Database & Models
- **Connection**: Always use `connectDB()` from `@/lib/db` before DB operations in API routes.
- **Models**: Define in `models/`. Use `mongoose.models.ModelName || mongoose.model("ModelName", Schema)` to prevent hot-reload errors.
- **User Schema**: Includes `provider` (google/credentials), `stats`, and `embeddingId`.

### Components & UI
- **UI Library**: Use `components/ui/` for atomic components (Button, Input, etc.).
- **Styling**: Use Tailwind classes. Merge classes with `cn()` utility (`lib/utils.js`).
- **Structure**: Feature components in `components/`, pages in `app/`.
- **Icons**: Use `lucide-react`.

### API Routes
- **Location**: `app/api/` (e.g., `app/api/movie/route.js`).
- **Response**: Return `NextResponse.json()`.
- **Error Handling**: Wrap DB calls in `try/catch` and return appropriate status codes.

### Client-Side Data
- **Caching**: Use `lib/idb.js` for caching large datasets or offline capability.
- **Fetching**: Prefer server components for initial data, client components for interactivity.

## Critical Workflows
- **Dev Server**: `npm run dev`
- **Linting**: `npm run lint`
- **Environment**: Ensure `.env` contains `MONGODB_URI`, `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`, `NEXTAUTH_SECRET`.

## Example: Database Connection in API Route
```javascript
import { connectDB } from "@/lib/db";
import User from "@/models/User";
import { NextResponse } from "next/server";

export async function GET() {
  try {
    await connectDB();
    const users = await User.find({});
    return NextResponse.json(users);
  } catch (error) {
    return NextResponse.json({ error: "Failed to fetch users" }, { status: 500 });
  }
}
```

## Example: UI Component Usage
```jsx
import { Button } from "@/components/ui/button";
import { cn } from "@/lib/utils";

export default function CustomAction({ className }) {
  return (
    <Button variant="destructive" className={cn("w-full", className)}>
      Delete Account
    </Button>
  );
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mannat244) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
