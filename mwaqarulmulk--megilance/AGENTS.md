# MegiLance AI Agent Instructions

> AI-powered freelancing platform: Next.js 16 + React 19 + FastAPI + Turso (libSQL)
not waste time on build use teh lci maxium like turso cli and other cli etc 
## Agent Work Standards (HIGH PRIORITY)

### Efficiency & Resource Management
- **Avoid redundant builds** - Don't waste time rebuilding Docker containers or recompiling unless necessary
- **Skip Docker for fast iteration** - Use direct `uvicorn` / `npm run dev` instead of `docker compose` during active development
- **Minimize automated script fixes** - Prioritize manual code review over blind automated fixes
- **Respect free tier limits** - Keep Turso/D1 database usage within free tier constraints (no billing upgrades)

### Progress Tracking & Task Execution
- Maintain detailed to-do lists for complex multi-step tasks
- Work iteratively until **100% completion** - no skipping or leaving work incomplete
- Continue autonomously without manual intervention
- Only request input for **critical reviews or decisions**, not routine problem-solving

### Problem-Solving Approach
- Identify and fix issues based on your understanding
- **Manually review all frontend elements** - examine CSS and TSX files for UI/UX, performance, security, and optimization issues
- Check all perspectives: layout, accessibility, responsiveness, dark/light themes
- **"Redesign" Definition**: When asked to "redesign", it means to **improve UI/UX** while strictly adhering to existing frontend architectures (Next.js 16, 3-file CSS modules). This implies a **from-scratch implementation** of the specific component/feature to ensure it meets modern standards, rather than patching existing code.

### Development Tools & Environment
- Prefer **CLI tools** for deployments, hosting, and database operations
- Extensively utilize **Chrome DevTools** for debugging and development tasks
- Use **MCP (Model Context Protocol)** development tools for AI-assisted development
- Execute all operations through **terminal/command prompt** interfaces
- Prioritize existing installed tools; install new ones only when absolutely necessary
- For authenticated services: verify auth status first, prompt for credentials if needed

### Work Execution Standards
- Maintain continuous automated operation without manual intervention
- Persistently work until all identified tasks and fixes are fully completed
- Provide progress updates only when critical issues arise or tasks are completed
- Ensure all plans and strategies are **100% fully implemented** - nothing skipped or left incomplete

## Architecture
```
Frontend (Next.js 16) → Backend (FastAPI) → Turso (libSQL)
                              ↓
                        File Storage (local/S3)
```
**Roles**: Client, Freelancer, Admin | **Auth**: JWT (30min access, 7 days refresh)

## Critical Patterns

### Frontend: 3-File CSS Module System (MANDATORY)
Every component requires exactly 3 CSS files:
```
Component.common.module.css  → layout, structure, animations
Component.light.module.css   → light theme colors only
Component.dark.module.css    → dark theme colors only
```
**Usage pattern** (see `frontend/app/components/Button/Button.tsx`):
```tsx
import { useTheme } from 'next-themes';
import { cn } from '@/lib/utils';
import commonStyles from './Component.common.module.css';
import lightStyles from './Component.light.module.css';
import darkStyles from './Component.dark.module.css';

const { resolvedTheme } = useTheme();
if (!resolvedTheme) return null; // Prevent flash
const themeStyles = resolvedTheme === 'light' ? lightStyles : darkStyles;
return <div className={cn(commonStyles.container, themeStyles.container)} />;
```

### Backend: Layered Architecture
```
Routers (api/v1/) → Services (services/) → Models/Schemas → DB Session
```
- **Routers**: HTTP handling only, call services
- **Services**: Pure business logic, no FastAPI imports
- **Schemas**: Split into request/response Pydantic models (`schemas/`)

### File Comments
Every file starts with `// @AI-HINT:` explaining its purpose:
```tsx
// @AI-HINT: Dual-flow payment wizard for withdrawals and adding funds
```

## Project Structure

### Frontend (`frontend/`)
```
app/
  (auth)/       → login, signup, forgot-password (public)
  (main)/       → marketing pages (public)
  (portal)/     → client/freelancer/admin dashboards (protected)
  components/   → reusable UI (Button, Input, UserAvatar)
  home/         → homepage sections
lib/utils.ts    → cn() class merge utility
styles/tokens.css → design tokens (colors, spacing)
```

### Backend (`backend/`)
```
app/
  api/v1/       → 30+ endpoint modules (auth, projects, proposals, payments...)
  core/         → config.py, security.py, rate_limit.py
  models/       → SQLAlchemy models (User, Project, Proposal, Contract...)
  schemas/      → Pydantic request/response schemas
  services/     → Business logic layer
  db/           → Database session, initialization
main.py         → App factory, middleware, startup
```

## Development Commands

```powershell
# Docker (recommended)
docker compose up -d                    # Start all services
docker compose -f docker-compose.dev.yml up --build  # Hot reload mode
docker compose logs -f backend          # View logs

# Backend standalone
cd backend
uvicorn main:app --reload --port 8000   # Dev server
pytest tests/ -v                        # Run tests
python comprehensive_test.py            # System smoke test

# Frontend standalone
cd frontend
npm run dev                             # Dev server :3000
npm test                                # Jest tests
```

**URLs**: Frontend `localhost:3000` | Backend API `localhost:8000/api/docs` | Health `localhost:8000/api/health/ready`

## API Conventions

- **Auth endpoints**: `/api/auth/register`, `/api/auth/login`, `/api/auth/refresh`, `/api/auth/me`
- **Resource pattern**: `GET /api/{resource}`, `POST /api/{resource}`, `GET /api/{resource}/{id}`
- **Frontend calls**: Use `/backend/api/*` proxy (configured in Next.js)
- **Error format**: `{"detail": "message", "error_type": "ExceptionType"}`
- **Rate limiting**: Enabled via `SLOWAPI_ENABLED` env var

## Key Conventions

### Button Component Contract
```tsx
// Variants: primary | secondary | danger | ghost | link | success | warning | social | outline
// Sizes: sm | md | lg | icon
<Button variant="primary" size="md" isLoading={false} fullWidth>Submit</Button>
<Button variant="social" provider="google">Sign in with Google</Button>
```

### Database
- **All environments**: Turso cloud database (libSQL) - REQUIRED
- **Setup**: Get free database at https://turso.tech
- **Config**: Set `TURSO_DATABASE_URL` and `TURSO_AUTH_TOKEN` in `.env`
- **Migrations**: `alembic revision --autogenerate -m "desc"` then `alembic upgrade head`

### Design Tokens
```css
Primary: #4573df | Success: #27AE60 | Error: #e81123 | Warning: #F2C94C | Accent: #ff9800
Fonts: Poppins (headings), Inter (body), JetBrains Mono (code)
```

## Common Pitfalls

❌ **Avoid**:
- Global CSS or inline styles (use 3-file CSS modules)
- Business logic in routers (use services layer)
- Missing `@AI-HINT` comments
- Hardcoded secrets (use `.env`)
- Breaking Button variant/size contracts

✅ **Always**:
- Test both light/dark themes
- Validate inputs via Pydantic
- Check `/api/health/ready` after backend changes
- Use `cn()` for class merging
- Add ARIA labels to interactive elements

## Key Documentation
- `docs/ENGINEERING_STANDARDS_2025.md` - Full coding standards
- `docs/Architecture.md` - System architecture details
- `docs/TURSO_SETUP.md` - Database configuration
- `frontend/README.md` - Frontend patterns
- `backend/README.md` - API documentation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mwaqarulmulk)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Mwaqarulmulk)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
