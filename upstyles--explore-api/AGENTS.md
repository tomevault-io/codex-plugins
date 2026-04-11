# Explore API — Copilot Instructions

## Mission
Build scalable content pipeline for UpStyles Explore feature supporting:
1. User-generated submissions with automated moderation
2. External content import (Instagram, TikTok, YouTube)
3. ML-powered personalized recommendations

## Architecture

- **Stack**: Node.js 20, TypeScript, Express
- **Module System**: ES Modules (`"type": "module"`) - ALWAYS include `.js` in local imports
- **Deployment**: Firebase App Hosting (or Cloud Run)
- **Auth**: Firebase ID tokens via `Authorization: Bearer <token>`
- **Database**: Firestore (submissions, user preferences, moderation queue)
- **Storage**: Cloud Storage (user-uploaded media)
- **Moderation**: Cloud Vision API (automated content safety)

## Project Structure

```
explore-api/
├── src/
│   ├── index.ts              # Express server + middleware
│   ├── routes/
│   │   ├── index.ts          # Route aggregator
│   │   └── submissions.ts    # Submission endpoints
│   ├── services/
│   │   ├── submissions.ts    # Business logic
│   │   └── moderation.ts     # Vision API integration
│   ├── middleware/
│   │   ├── auth.ts           # Firebase Auth verification
│   │   └── rate-limit.ts     # Rate limiting logic
│   └── lib/
│       ├── firebase.ts       # Firebase Admin initialization
│       └── validation.ts     # Zod schemas
└── package.json
```

## Key Conventions

### 1. Import Syntax
**ALWAYS** include `.js` extension for local imports (ES Modules requirement):
```typescript
// ✅ Correct
import { getFirestore } from './firebase.js';
import { SubmissionService } from '../services/submissions.js';

// ❌ Wrong
import { getFirestore } from './firebase';
```

### 2. Request Validation
Use Zod schemas from `lib/validation.ts` for all input validation:
```typescript
import { SubmissionSchema } from '../lib/validation.js';

router.post('/submissions', async (req, res) => {
  const input = SubmissionSchema.parse(req.body); // Throws on invalid
  // ... process
});
```

### 3. Authentication Pattern
```typescript
import { authenticateUser, AuthenticatedRequest } from '../middleware/auth.js';

router.post('/endpoint', authenticateUser, async (req: AuthenticatedRequest, res) => {
  const userId = req.user!.uid; // Guaranteed to exist after auth middleware
});
```

### 4. Rate Limiting
Apply appropriate rate limiters per endpoint type:
```typescript
import { rateLimitSubmissions, rateLimitAPI } from '../middleware/rate-limit.js';

// High-cost operations (submissions)
router.post('/submissions', authenticateUser, rateLimitSubmissions, handler);

// General endpoints
app.use(rateLimitAPI); // Applied globally
```

### 5. Error Handling
```typescript
try {
  // ... business logic
} catch (error: any) {
  if (error.name === 'ZodError') {
    return res.status(400).json({ error: 'Invalid input', details: error.errors });
  }
  console.error('[Service] Error:', error);
  res.status(500).json({ error: 'Internal server error' });
}
```

## Firestore Collections

### `explore_submissions`
User-generated submissions awaiting moderation
```typescript
{
  userId: string;
  status: "pending" | "approved" | "rejected" | "flagged";
  type: "design" | "technique" | "product" | "tutorial";
  title: string;
  description: string;
  mediaUrls: string[];
  tags: string[];
  difficulty?: string;
  priceRange?: string;
  materials?: string[];
  moderationFlags: {
    spam: number;         // 0-1 confidence
    inappropriate: number;
    aiGenerated: boolean;
  };
  submittedAt: Timestamp;
  reviewedAt?: Timestamp;
  reviewedBy?: string;
  rejectionReason?: string;
  approvedEntryId?: string;
}
```

### `user_explore_preferences`
User behavior tracking for ML recommendations (Phase 3)
```typescript
{
  userId: string;
  interests: string[];
  engagementHistory: {
    viewedEntries: string[];
    savedEntries: string[];
    sharedEntries: string[];
  };
  mlProfile?: {
    embeddings: number[];
    lastUpdated: Timestamp;
  };
}
```

## Content Moderation Flow

1. User submits content → `POST /api/explore/submissions`
2. Upload media to Cloud Storage (if not already uploaded)
3. Run `moderateSubmission()` from `services/moderation.ts`:
   - Check rapid submission pattern (spam detection)
   - Scan all images via Vision API
   - Detect adult/violence/racy content
4. Determine status:
   - `safe` → status: `"pending"`
   - `flagged` → status: `"flagged"` (manual review)
5. Store submission in Firestore
6. Return submission ID + status to client

## Rate Limiting Strategy

- **Submissions**: 10/day per user (prevent spam)
- **General API**: 60/minute per IP (DDoS protection)
- **External Sync**: 1/hour per platform (API quota management)

Use `rate-limiter-flexible` library with in-memory storage (Phase 1).
For production scale, switch to Redis backend.

## Adding New Endpoints

1. **Create route handler** in `src/routes/`:
   ```typescript
   import { Router } from 'express';
   export default router;
   ```

2. **Define Zod schema** in `src/lib/validation.ts`:
   ```typescript
   export const MyInputSchema = z.object({ ... });
   ```

3. **Implement service** in `src/services/`:
   ```typescript
   export class MyService { ... }
   ```

4. **Register route** in `src/routes/index.ts`:
   ```typescript
   import myRouter from './my-route.js';
   router.use('/my-path', myRouter);
   ```

## External Integrations (Phase 2)

### Instagram
1. User initiates OAuth flow in Flutter app
2. Flutter sends auth code to `/api/explore/connect/instagram`
3. Backend exchanges code for access token
4. Fetch recent posts via Instagram Graph API
5. Create submissions (pre-moderated) for each post
6. Store connection details in `user_explore_preferences`

### TikTok
Similar flow using TikTok Content Posting API.

## ML Recommendations (Phase 3)

### Training Pipeline
1. Export engagement data to BigQuery
2. Train embeddings model on Vertex AI
3. Generate user preference vectors
4. Store in `user_explore_preferences.mlProfile`

### Inference Endpoint
```typescript
GET /api/explore/feed/personalized?audience=enthusiast&limit=20

// Returns sections with personalized trendScore per user
```

## Testing Locally

```bash
# Install dependencies
npm install

# Start emulators (from upstyles_app/)
./scripts/start_emulators.sh

# Run dev server
npm run dev

# Test health endpoint
curl http://localhost:8080/api/health

# Test submission (requires valid Firebase token)
curl -X POST http://localhost:8080/api/explore/submissions \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"type":"design","title":"Test","mediaUrls":["https://example.com/img.jpg"],"tags":["test"]}'
```

## Deployment

### Firebase App Hosting
```bash
# One-time setup
firebase apphosting:backends:create explore-api

# Subsequent deploys (auto-triggered on git push)
git push origin main
```

### Cloud Run (Alternative)
```bash
gcloud run deploy explore-api \
  --source . \
  --region us-east4 \
  --allow-unauthenticated
```

## Common Patterns

### Pagination
```typescript
const query = db.collection('explore_submissions')
  .orderBy('submittedAt', 'desc')
  .limit(limit + 1); // Fetch one extra to check hasMore

if (cursor) {
  const cursorDoc = await db.collection('explore_submissions').doc(cursor).get();
  query = query.startAfter(cursorDoc);
}

const snapshot = await query.get();
const docs = snapshot.docs.slice(0, limit);
const hasMore = snapshot.docs.length > limit;

return {
  items: docs.map(doc => doc.data()),
  nextCursor: hasMore ? docs[docs.length - 1].id : undefined,
};
```

### Role-Based Access
```typescript
import { requireRole } from '../middleware/auth.js';

// Only moderators can access
router.get('/moderation/queue', authenticateUser, requireRole('moderator'), handler);
```

### Firestore Transactions
For critical writes (e.g., approval):
```typescript
await db.runTransaction(async (transaction) => {
  const submissionRef = db.collection('explore_submissions').doc(submissionId);
  const entryRef = db.collection('explore_collections').doc(collectionId).collection('entries').doc();
  
  transaction.update(submissionRef, { status: 'approved' });
  transaction.set(entryRef, entryData);
});
```

## Monitoring

Key metrics to track:
- Submission creation rate (per hour)
- Moderation queue depth
- Vision API cost (per request)
- Approval/rejection ratio
- API error rate

Set up Cloud Monitoring alerts for:
- Queue depth > 100
- API error rate > 1%
- Vision API quota > 80%

## Security Checklist

- ✅ All endpoints require Firebase Auth (except `/health`)
- ✅ Rate limiting on submissions (10/day)
- ✅ Content moderation before publishing
- ✅ Role-based access for moderation endpoints
- ✅ Input validation with Zod schemas
- ✅ Firestore rules restrict direct client writes

## Troubleshooting

**"Cannot find module" errors**
→ Check `.js` extensions in imports

**"Rate limit exceeded"**
→ Check `RATE_LIMIT_SUBMISSIONS_DAILY` in `.env`

**"Vision API quota exceeded"**
→ Increase quota or implement caching

**"Firebase Admin not initialized"**
→ Check `FIREBASE_PROJECT_ID` and credentials path

## References

- Main architecture: `../upstyles_app/EXPLORE_API_ARCHITECTURE.md`
- Implementation guide: `../upstyles_app/EXPLORE_IMPLEMENTATION_SUMMARY.md`
- Flutter integration: `../upstyles_app/lib/src/services/explore_api_service.dart`
- Similar patterns: `../chat-api/`, `../map-api/`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/upstyles)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/upstyles)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
