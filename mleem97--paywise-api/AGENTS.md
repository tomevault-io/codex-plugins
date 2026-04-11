# Copilot Instructions for paywise-api

> **Maintainer:** [@mleem97](https://github.com/mleem97)  
> **Organization:** Lunexor / MVNet Solutions / Meyer Media  
> **NPM:** https://www.npmjs.com/package/paywise-api  
> **Docs:** https://docs.paywise.de

## ⚠️ MANDATORY: Before ANY Code Change

1. **Check current version**: `npm view paywise-api version` or check [NPM](https://www.npmjs.com/package/paywise-api)
2. **All code MUST**: Build (`npm run build`), Lint (`npm run lint`), Test (`npm run test`)
3. **Commit with semantic prefix** → triggers automatic versioning
4. **Update README.md** if API surface changes
5. **CHANGELOG.md** follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/) format

## Architecture

```
src/
├── index.ts              # Main exports
├── client.ts             # PaywiseClient (main entry point)
├── http-client.ts        # HTTP layer with fetch, retry, rate limiting
├── types.ts              # Shared types (PaywiseConfig, ApiResponse, ApiError)
├── case-management/      # Case Management API module
│   ├── client.ts         # CaseManagementClient
│   └── types.ts          # Type definitions (~850 lines)
└── partner-api/          # Partner API module
    ├── client.ts         # PartnerApiClient (requires X-User-Id header)
    └── types.ts          # Partner-specific types
tests/
└── mocks/                # Mock API responses based on docs.paywise.de
```

## Semantic Versioning (MANDATORY)

Every change MUST be committed with proper semantic prefix:

| Prefix | Version Bump | Example |
|--------|--------------|---------|
| `fix:` | Patch (1.0.X) | `fix: handle 429 rate limit errors` |
| `feat:` | Minor (1.X.0) | `feat: add batch claim creation` |
| `feat!:` or `BREAKING CHANGE:` | Major (X.0.0) | `feat!: change Amount type structure` |
| `docs:` | No release | `docs: update README examples` |
| `chore:` | No release | `chore: update dependencies` |
| `test:` | No release | `test: add mock API tests` |

**Before pushing**: Verify no duplicate versions exist on NPM!

## API Type Conventions

```typescript
// ✅ CORRECT: snake_case properties, index signature, Amount type
export interface CreateClaimParams {
  [key: string]: unknown;
  debtor: string;
  total_claim_amount: Amount;  // { value: string; currency: 'EUR' }
  due_date: string | null;
}

// ❌ WRONG: camelCase won't match Paywise API
export interface CreateClaimParams {
  debtorId: string;
  totalAmount: number;
}
```

## Critical Edge Cases to Handle

### Authentication
- Token expiration during multi-step operations → Implement retry with backoff
- Partner API requires `X-User-Id` header for company context

### Rate Limiting
- API may return 429 → Exponential backoff: 1s, 2s, 4s, 8s...
- Implement client-side rate limiter for bulk operations

### Claim Workflow
- **NEVER release claim without document upload** → Pre-validation required
- Debtor duplicate detection → Provide `findOrCreateDebtor()` helper
- Mandate acceptance requires polling → `waitForMandateAcceptance()` helper

### File Handling
- Browser: `File` API
- Node.js: `Buffer` + metadata OR `File` (Node 20+)
- Validate MIME types before upload

## Commands

```bash
npm run build       # TypeScript compilation
npm run lint        # ESLint v9 (entire workspace)
npm run lint:fix    # Auto-fix linting issues
npm run test        # Jest with mock APIs
npm run prepare     # Husky git hooks
```

## Testing Requirements

Tests use **mock APIs** based on [docs.paywise.de](https://docs.paywise.de):

```typescript
// tests/mocks/claim.mock.ts
export const mockClaimResponse: Claim = {
  id: 'claim-123',
  href: '/v1/claims/claim-123',
  debtor: '/v1/debtors/debtor-456',
  // ... based on actual API response schema
};
```

Test categories:
- Unit tests for type validation
- Integration tests with mocked HTTP responses
- Edge case tests (timeout, 429, network failures)

## Key Files

| File | Purpose |
|------|---------|
| `src/case-management/types.ts` | All Case Management types |
| `src/partner-api/types.ts` | Partner API types |
| `scripts/update-changelog.js` | Keep a Changelog formatter |
| `eslint.config.mjs` | ESLint v9 flat config |
| `.github/workflows/release.yml` | CI/CD with NPM OIDC |
| `tests/mocks/` | Mock API responses |

## Adding New Features

1. Check [docs.paywise.de](https://docs.paywise.de) for API spec
2. Add types to `src/{module}/types.ts` (snake_case!)
3. Add method to `src/{module}/client.ts`
4. Add mock to `tests/mocks/`
5. Write tests
6. Export from `src/index.ts`
7. Update `README.md` with usage example
8. Run `npm run build && npm run lint && npm run test`
9. Commit with semantic prefix

## Common Pitfalls

- **Type errors**: Add `[key: string]: unknown` index signature
- **Partner API 403**: Missing `X-User-Id` header
- **Release failed**: Check NPM for existing version, push tags with `git push --tags`
- **Claim release rejected**: Upload document first!
- **IBAN validation**: Consider `ibantools` for client-side validation
- **Semantic-release versioning**: Tags must exist on remote; use `git push --tags`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mleem97)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/mleem97)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
