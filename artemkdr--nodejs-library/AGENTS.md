# GitHub Copilot Instructions for Node.js library

## Project Overview

This is a TypeScript-based Node.js library unifying different packages that will be used over multiple projects.
This packages will be published to GitHub Package Registry.

## Technologies 

- **Package Manager**: Bun
- **Language**: TypeScript (strict mode)

## TypeScript Development Standards

### Strict TypeScript Configuration

```typescript
// Always use strict TypeScript settings
{
  "strict": true,
  "noImplicitAny": true,
  "strictNullChecks": true,
  "strictFunctionTypes": true,
  "noImplicitReturns": true,
  "noUnusedLocals": true,
  "noUnusedParameters": true
}
```

### Code Quality Rules

1. **No `any` type**: Always provide explicit types
2. **Strict null checks**: Handle null/undefined explicitly
3. **Immutable patterns**: Prefer readonly and const assertions
4. **Error handling**: Use Result/Either patterns or proper error types
5. **Dependency injection**: Use interfaces for loose coupling
6. **Split into pieces**: Break down large files into smaller, focused modules if needed. Ideally one file per class /  service. Look at the existing directory structure for guidance.
7. **Keep it simple**: Do not over-engineer solutions. Aim for simplicity and clarity. No functions for "just in case" scenarios.

## Architecture Patterns

### Loose Coupling Principles

1. **Dependency Inversion**: Depend on abstractions, not concretions
2. **Interface Segregation**: Create focused, single-purpose interfaces
3. **Single Responsibility**: Each module has one reason to change
4. **Repository Pattern**: Abstract data access behind interfaces

### Logging

- No logging as it's a libraries repository.

### Testing Requirements

For **all**:
- Do not test logging outputs

For **new code**:
- Minimum 90% code coverage
- Test all public methods
- Test error conditions
- Mock external dependencies

For **refactoring/fixes**:
- Ensure existing tests still pass
- Add tests for previously uncovered scenarios
- Update tests that validate changed behavior

#### Testing Patterns

```typescript
// Use dependency injection for testability
describe('SearchService', () => {
  let service: SearchService;
  let mockRepo: jest.Mocked<IRepository>;
  let mockVectorStore: jest.Mocked<IVectorStore>;

  beforeEach(() => {
    mockRepo = createMockRepository();
    mockVectorStore = createMockVectorStore();
    service = new SearchService(mockRepo, mockVectorStore);
  });

  describe('search', () => {
    it('should function correctly', async () => {
      // test main functionality
    });

    it('should handle empty results gracefully', async () => {
      // Test edge cases
    });

    it('should handle service errors', async () => {
      // Test error conditions
    });
  });
});
```

## Development Workflow

When adding or modifying any code, **ALWAYS** follow these steps:

### 1. Build a Plan

Before implementing, create a detailed plan that includes:

- **Objective**: What specific functionality are we implementing?
- **Components**: Which files/modules will be affected?
- **Interfaces**: What new types/interfaces are needed?
- **Dependencies**: What external services or modules are required?
- **Testing Strategy**: What needs to be tested and how?
- **Breaking Changes**: Will this affect existing APIs or behavior?

**Example Plan Format:**
```markdown
## Implementation Plan

### Objective
Implement hybrid search endpoint combining semantic and metadata filtering

### Components Affected
- `src/api/routes/search.ts` (new)
- `src/search/hybrid.ts` (new)
- `src/types/search.ts` (modify)

### New Interfaces
- `SearchRequest`, `SearchResponse`, `HybridSearchOptions`

### Dependencies
- LangChain VectorStore
- PostgreSQL for metadata filtering
- OpenAI embeddings service

### Testing Strategy
- Unit tests for search logic
- Integration tests for API endpoints
- Mock external dependencies
```

### 2. Start Implementation After User's Confirmation

- Present the plan to the user
- Wait for explicit confirmation before proceeding
- Ask for clarification on any ambiguous requirements

### 3. Implement

Proceed with implementation according to the plan.

### 4. Cover with Unit Tests

Add unit tests for all new functionality, including edge cases and error conditions.

### 5. Output Summary of Changes

After implementation, provide a comprehensive summary:

#### Summary Format
```markdown
## Implementation Summary

### Files Added/Modified
- ✅ `src/search/hybrid.ts` - New hybrid search service
- ✅ `src/api/routes/search.ts` - Search API endpoints
- ✅ `src/types/search.ts` - Search-related type definitions
- ✅ `tests/search/hybrid.test.ts` - Unit tests for hybrid search
- ✅ `tests/api/search.integration.test.ts` - Integration tests

### Key Features Implemented
- Hybrid search combining semantic and metadata filtering
- Type-safe API with comprehensive error handling
- Caching layer for improved performance
- Comprehensive test coverage (90%)

### Breaking Changes
- None

### Next Steps
- [ ] Add query performance monitoring
- [ ] Implement search analytics
- [ ] Add more sophisticated ranking algorithms

### Testing Results
- Unit tests: 23 passed, 0 failed
- Integration tests: 5 passed, 0 failed
- Coverage: 93% (exceeds 90% requirement)
```

## Domain-Specific Guidelines

### API Design
- Follow RESTful conventions
- Implement proper HTTP status codes
- Provide comprehensive error messages
- Version APIs for backward compatibility

## Input Validation

- Implement comprehensive input validation using Zod schemas
- Ensure all user inputs are validated before processing
- Provide clear error messages for invalid inputs

## Security Guidelines

- Validate all input data
- Sanitize database queries
- Implement proper authentication
- Never expose internal error details in API responses

Remember: Every piece of code should be type-safe, testable, and loosely coupled. When in doubt, favor explicit types and clear interfaces over clever implementations.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/artemkdr)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/artemkdr)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
