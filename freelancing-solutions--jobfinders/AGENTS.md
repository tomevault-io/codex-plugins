# AI Agent Instructions for JobFinders

## Work Mode: Agentic Development

As an AI agent working on this project, you should:
1. Work autonomously on feature implementation
2. Only ask questions after feature completion and testing
3. Request human review for:
   - Completed feature implementations
   - New documentation
   - Specification files
   - Architecture changes
4. Make independent decisions about implementation details within established patterns
5. Follow existing conventions and patterns in the codebase

## Project Overview
JobFinders is a Next.js-based job board platform with AI-powered features using OpenRouter API. The application serves both job seekers and employers with features like AI resume building, ATS analysis, and intelligent job matching.

## Key Architectural Patterns

### AI Services Layer
- All AI interactions go through `src/lib/openrouter.ts`
- Use claude-2 for complex tasks, gpt-3.5-turbo for basic interactions
- Always implement rate limiting and fallback handling
```typescript
// Example pattern from src/lib/openrouter.ts
const ai = new OpenRouterClient({
  model: 'claude-2',
  fallback: 'gpt-3.5-turbo'
});
```

### Data Flow
1. Client Request → API Routes (`src/app/api/*`)
2. API Routes → Service Layer (`src/services/*`)
3. Services → OpenRouter/DB/External APIs
4. Response → Client

### Authentication & Authorization
- NextAuth.js with custom middleware
- Role-based access (seeker, employer, admin)
- Subscription-based feature gating

## Project Structure
```
src/
├── app/          # Next.js App Router pages & API routes
├── services/     # Business logic & AI service implementations
├── components/   # React components (heavy use of shadcn/ui)
└── lib/          # Shared utilities & configurations
```

## Development Workflows

### Local Development
```bash
npm install        # Install dependencies
npm run dev       # Start development server
npm run build     # Production build
npm run test      # Run test suite
```

### Database Operations
- Use Prisma for all database operations
- Always generate types after schema changes:
```bash
npx prisma generate
npx prisma db push  # Development
npx prisma migrate deploy  # Production
```

## Critical Patterns

### AI Feature Implementation
1. Define service in `src/services`
2. Create API route in `src/app/api`
3. Implement rate limiting
4. Add subscription checks
5. Handle errors & fallbacks

Example:
```typescript
export class AIService {
  private ai: OpenRouterClient;
  
  async process(input: string) {
    try {
      await this.checkRateLimit();
      return await this.ai.chat(input);
    } catch {
      return await this.fallback(input);
    }
  }
}
```

### Subscription Handling
- Check subscription status in API routes
- Use feature flags for premium features
- Handle PayPal webhook events

### Error Handling
- Use custom error classes from `src/lib/errors`
- Always implement fallback behavior for AI features
- Log errors with context for debugging

## Integration Points

### External Services
- OpenRouter API for AI features
- PayPal for payments
- PostgreSQL for data storage
- Redis for caching & rate limiting

### Cross-Component Communication
- Use React Context for global state
- Socket.IO for real-time features
- Redis pub/sub for background jobs

## Testing Guidelines
- Unit tests for services
- Integration tests for API routes
- E2E tests for critical user flows

## Performance Considerations
- Implement caching for AI responses
- Use optimistic updates for UI
- Lazy load components and features

## Documentation
- API documentation in `/docs/api`
- AI feature specs in `/docs/specs`
- Architecture decisions in `/docs/ARCHITECTURE.md`

## Development Process

### Task-Based Workflow
1. Before starting work, check `/docs/TODO-*.md` files for existing tasks
2. Each implementation task should be marked with:
   ```markdown
   - [ ] Task description (status: not started)
   - [~] Task in progress (status: implementing)
   - [x] Task completed (status: done, date: YYYY-MM-DD)
   ```
3. Update task status in the appropriate TODO file as you progress
4. Add implementation notes under completed tasks for future reference:
   ```markdown
   - [x] Implement PayPal subscription handling (status: done, date: 2025-09-03)
     - Added webhook controller in `src/app/api/webhooks/paypal/route.ts`
     - Implemented subscription service in `src/services/subscription.ts`
     - Added test cases in `__tests__/subscription.test.ts`
   ```

### Agentic Implementation
1. Read and understand relevant spec files
2. Implement features autonomously following established patterns
3. Write tests and ensure passing
4. Update documentation as needed
5. Request review only when feature is complete and tested

### When to Seek Input
- After completing a feature implementation
- When creating new documentation
- When developing new spec files
- When proposing architecture changes
- When suggesting new project-wide patterns

### Independent Decision Making
- Implementation details within established patterns
- Choice of utility functions and helpers
- Component structure and organization
- Test coverage approach
- Performance optimizations

### Task Management Best Practices
1. Always check existing TODO files before starting new work
2. Keep task status updated in real-time
3. Add detailed implementation notes for completed tasks
4. Link related tasks together with references
5. If creating new tasks, follow the established format
6. Use consistent status markers ([ ], [~], [x])
7. Include dates for task completion tracking

Remember: The goal is to work independently while maintaining consistency with the project's architecture and patterns. Documentation and specifications should be comprehensive enough to enable autonomous development. Always update task status to prevent duplicate work and maintain clear progress tracking.

---
> Source: [freelancing-solutions/jobfinders](https://github.com/freelancing-solutions/jobfinders) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-22 -->
