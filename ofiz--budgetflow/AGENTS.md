# Budget App - AI Coding Agent Instructions

## Project Overview
Budget app is a greenfield project with skill-based development guidelines. This workspace uses a modular skill system ([.github/skills/](.github/skills/)) defining best practices for different development aspects.

## Architecture & Structure

### Expected Structure (not yet implemented)
Based on the skill files, this project should follow:
- **Backend**: RESTful API with Python (FastAPI/Django) or TypeScript backend
- **Frontend**: Modern UI with distinctive design (avoid generic aesthetics)
- **Database**: SQL with UUID public IDs, soft deletes, proper indexing
- **Testing**: Playwright for E2E tests, pytest/jest for unit tests

### Key Skills Applied Project-Wide

All skills in [.github/skills/](.github/skills/) are active:
1. **[Security (OWASP)](.github/skills/security-owasp/SKILL.md)** - Applied to ALL code (`applyTo: '*'`)
2. **[Backend Development](.github/skills/backend-development/SKILL.md)** - API design, database patterns
3. **[Python Development](.github/skills/python-development/SKILL.md)** - Modern Python 3.12+ patterns
4. **[Frontend Design](.github/skills/frontend-design/SKILL.md)** - Distinctive UI (avoid Inter/Roboto fonts)
5. **[Web App Testing](.github/skills/webapp-testing/SKILL.md)** - Playwright testing
6. **[Code Refactoring](.github/skills/code-refactoring/SKILL.md)** - Refactoring patterns

## Critical Security Requirements (OWASP)

**Security is non-negotiable** - All code must follow OWASP Top 10 mitigations:

### Always Enforce
- **Access Control**: Deny by default, least privilege, prevent path traversal
- **Secrets**: Never hardcode - use `process.env` or secrets manager
- **SQL Injection**: Use parameterized queries exclusively
- **XSS**: Use `.textContent` over `.innerHTML`; sanitize with DOMPurify if needed
- **HTTPS**: All external requests use HTTPS
- **Password Hashing**: Argon2 or bcrypt only (never MD5/SHA-1)
- **Session Security**: `HttpOnly`, `Secure`, `SameSite=Strict` cookies

### When Suggesting Code
Explicitly state security protections (e.g., "Using parameterized query to prevent SQL injection").

## Development Patterns

### API Design (from backend-development skill)
```
GET    /api/resource       # List
POST   /api/resource       # Create
GET    /api/resource/:id   # Get
PATCH  /api/resource/:id   # Update
DELETE /api/resource/:id   # Delete
```

**Response format:**
```json
{
  "data": { ... },
  "meta": { "page": 1, "per_page": 20, "total": 100 }
}
```

### Database Conventions
```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  public_id UUID DEFAULT gen_random_uuid() UNIQUE,
  email VARCHAR(255) UNIQUE NOT NULL,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  deleted_at TIMESTAMPTZ  -- Soft deletes
);
```

### Frontend Design Philosophy
- **Avoid generic AI aesthetics**: No Inter, Roboto, Arial fonts
- **Typography**: Choose distinctive fonts from Google Fonts
- **Color**: Cohesive palettes with CSS variables, dominant colors + sharp accents
- **Layout**: Asymmetry, overlap, diagonal flow - break predictable grids
- **Motion**: High-impact orchestrated animations > scattered micro-interactions

### Python Patterns (Python 3.12+)
```python
# Modern type hints
def process(items: Sequence[str]) -> list[str]: ...

# Async operations
async with aiohttp.ClientSession() as session:
    tasks = [fetch_one(session, url) for url in urls]
    return await asyncio.gather(*tasks)

# Pydantic for data validation
class UserCreate(BaseModel):
    email: str
    name: str
```

## Testing Strategy

### Playwright E2E Tests
```typescript
import { test, expect } from '@playwright/test';

test('user can add budget entry', async ({ page }) => {
  await page.goto('http://localhost:3000');
  await page.fill('[data-testid="amount"]', '100');
  await page.click('[data-testid="submit"]');
  await expect(page.locator('.budget-list')).toContainText('$100');
});
```

### Test Commands (when implemented)
- `npx playwright test` - Run E2E tests
- `pytest` - Run Python unit tests
- `npm test` - Run JavaScript/TypeScript tests

## Refactoring Guidelines

### When to Refactor
- Before adding features (make change easy, then make easy change)
- After tests pass (red-green-refactor cycle)
- During code review

### Common Patterns
- **Long methods**: Extract focused methods
- **Deep nesting**: Use guard clauses (early returns)
- **Primitive obsession**: Create value objects
- **Magic numbers**: Extract named constants

## Commands & Workflows

Since no `package.json` or `pyproject.toml` exists yet, typical commands will be:

### Python Backend (when created)
```bash
python -m venv venv
. venv/Scripts/activate  # Windows PowerShell
pip install -r requirements.txt
python -m uvicorn main:app --reload
```

### Frontend (when created)
```bash
npm install
npm run dev
npm run build
```

### Testing
```bash
npx playwright test --ui
pytest tests/ -v
```

## Project-Specific Conventions

1. **UUID Public IDs**: Use UUIDs for external-facing IDs, serial for internal
2. **Soft Deletes**: Add `deleted_at` timestamp instead of hard deletes
3. **Security First**: Always choose secure option, explain reasoning
4. **Distinctive Design**: Commit to bold aesthetic direction
5. **Type Safety**: Use TypeScript strict mode or Python mypy strict

## File Organization (Suggested)

```
budget_app/
├── backend/               # API server
│   ├── app/
│   │   ├── models/       # Database models
│   │   ├── routes/       # API endpoints
│   │   └── middleware/   # Auth, rate limiting
│   └── tests/
├── frontend/             # UI application
│   ├── src/
│   │   ├── components/
│   │   ├── pages/
│   │   └── styles/
│   └── tests/
├── .github/
│   └── skills/           # Skill-based guidelines
└── README.md
```

## Getting Started with New Features

When implementing features:
1. **Check security requirements** in [security-owasp skill](.github/skills/security-owasp/SKILL.md)
2. **Follow API patterns** from [backend-development skill](.github/skills/backend-development/SKILL.md)
3. **Apply design principles** from [frontend-design skill](.github/skills/frontend-design/SKILL.md)
4. **Write tests** using [webapp-testing skill](.github/skills/webapp-testing/SKILL.md) patterns
5. **Refactor progressively** using [code-refactoring skill](.github/skills/code-refactoring/SKILL.md)

## Important Notes

- All skills are authoritative - follow their patterns precisely
- Security skill (`applyTo: '*'`) applies to ALL code generation
- Avoid suggesting generic/boilerplate solutions - reference specific skill patterns
- When uncertain about architecture, ask before implementing

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ofiz) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
