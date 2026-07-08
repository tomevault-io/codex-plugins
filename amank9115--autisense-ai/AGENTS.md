# Copilot Instructions for AutiSense-AI

## Project Overview

**AutiSense-AI** is an AI-enabled autism screening platform with real-time behavioral analysis, ML risk prediction, and clinical-grade reports. It uses a three-tier architecture:

- **Frontend**: Next.js 15 (React 19) with Turbopack
- **Backend**: NestJS with Express, Prisma ORM, PostgreSQL
- **ML Service**: FastAPI with scikit-learn, OpenCV, MediaPipe

Services run concurrently on **localhost:3000** (frontend), **localhost:4000** (backend), and **localhost:8001** (ML service).

## Build, Test & Lint Commands

### Root Level
```bash
npm run dev               # Start all three services with concurrently
npm run install:all      # Install dependencies for all services
```

### Backend (NestJS)
```bash
cd backend
npm run dev              # Start with hot-reload (nodemon)
npm run build            # Compile to dist/
npm run lint             # Fix ESLint violations
npm run test             # Run Jest unit tests
npm run test:watch       # Watch mode for tests
npm run test:cov         # Generate coverage report
npm run test:e2e         # Run end-to-end tests
npm run format           # Format with Prettier
```

### Frontend (Next.js)
```bash
cd frontend
npm run dev              # Start dev server with Turbopack
npm run build            # Production build
npm run lint             # Run ESLint (does not auto-fix)
npm start                # Start production server
```

### ML Service (FastAPI)
```bash
cd ml-service
# Activate venv first on Windows:
.\\venv\\Scripts\\activate
# Then:
uvicorn app.main:app --host 127.0.0.1 --port 8001 --reload
```

## Architecture

### Data Flow
1. **Frontend** (Next.js) sends requests to backend API routes that rewrite to `/api/v1`
2. **Backend** (NestJS) routes requests to appropriate services (auth, screening, AI, etc.)
3. **ML Service** (FastAPI) handles camera analysis, emotion detection, and PDF generation
4. **Database** (PostgreSQL) via Prisma for users, children, sessions, and metadata

### Key Modules

**Backend Structure** (`backend/src/`):
- `auth/` — JWT authentication, password hashing (bcryptjs)
- `users/` — User management and role-based access control
- `screening/` — Screening session creation and management
- `ml/` — Proxy gateway to FastAPI ML service
- `ai/` — Langchain integration with Gemini API for AI assistant
- `queue/` — BullMQ for async task processing (emails, reports)
- `email/` — Email service via Resend
- `prisma/` — Database schema and migrations

**Frontend Structure** (`frontend/src/`):
- `app/` — Next.js app directory routes
- `api/` — API client and data fetching logic
- `components/` — React components organized by feature
- `context/` — React context for auth, theme, screening state
- `services/` — HTTP clients and service layer
- `store/` — Zustand state management
- `types/` — TypeScript type definitions

**ML Service Structure** (`ml-service/`):
- `app/main.py` — FastAPI entry point
- `app/ml_analyzer.py` — ML inference and scoring
- `app/pdf_generator.py` — ReportLab for PDF generation
- `ai-engine/` — MediaPipe, OpenCV, emotion detection
- `models/` — Pre-trained .pkl files (in .gitignore)

## Key Conventions

### Code Style
- **Prettier config** (backend): Single quotes, trailing commas
- **ESLint**: Strict mode enabled in both frontend and backend
- **TypeScript**: Strict type checking enabled (noImplicitAny, strictNullChecks, etc.)

### Naming & Structure
- **NestJS modules** follow feature-based structure (auth.module.ts, users.module.ts)
- **DTOs** (Data Transfer Objects) for request/response validation with class-validator
- **Services** contain business logic; controllers handle HTTP layer
- **Decorators**: @Controller, @Service, @Module for NestJS; @Get, @Post for routes

### Database
- **Prisma** schema defines models; migrations are auto-generated
- **PostgreSQL** is the production database (see DATABASE_URL in .env)
- **Enums** (Role, etc.) defined in Prisma schema
- Relations use explicit foreign keys with @relation decorators

### Authentication
- **JWT** tokens issued on login; validated via passport-jwt strategy
- **Passwords** hashed with bcryptjs; never stored in plaintext
- **Role-based access control** (RBAC): parent, doctor roles

### ML Integration
- **PY_ML_ENABLED** and **AI_ENGINE_ENABLED** env vars control gateway features
- **FastAPI** endpoints proxied through `/api/v1/ml/*` routes
- **Timeouts**: 2500ms for ML service, 3500ms for AI engine

### Environment Variables
- Backend: `DATABASE_URL`, `PORT`, `MONGODB_URI` (legacy), `GEMINI_API_KEY`
- Frontend: `NEXT_PUBLIC_API_BASE_URL` (must be prefixed `NEXT_PUBLIC_` to be visible in browser)
- ML Service: Configured via `pyproject.toml` (Python 3.10+)

### Testing
- **Jest** for backend unit tests (*.spec.ts files in src/)
- **E2E tests** use supertest for HTTP endpoint testing
- **Test regex**: `.*\\.spec\\.ts$` matches test files
- **Passthrough mode**: `npm run test -- --passWithNoTests` to pass even if no tests exist

## Development Workflow

1. **Starting Development**:
   ```bash
   npm run install:all      # First time only
   npm run dev              # Starts all services concurrently
   ```

2. **Making Backend Changes**:
   - Edit TypeScript in `backend/src/`
   - NestJS auto-reloads with --watch flag
   - Run `npm run lint` to check ESLint rules
   - Run `npm run test:watch` for TDD

3. **Making Frontend Changes**:
   - Edit Next.js components in `frontend/src/`
   - Dev server auto-reloads via Turbopack
   - Run `npm run lint` to check (doesn't auto-fix; requires manual edits)

4. **Database Migrations**:
   ```bash
   cd backend
   npx prisma migrate dev --name migration_name   # Create and apply
   npx prisma studio                               # GUI for database
   ```

5. **Python ML Development**:
   - Edit `.py` files in `ml-service/`
   - Uvicorn reloads with `--reload` flag
   - Test with `python -m py_compile app/main.py`

## CI Pipeline

The GitHub Actions workflow (`.github/workflows/ci.yml`) runs:
- **Backend**: Build → Lint → Test → Generate Prisma client
- **Frontend**: Build → Lint (CI only, doesn't fix)
- **ML Service**: Compile Python modules

All three jobs run in parallel on `push` to main and on pull requests. Backend uses Node 22, frontend uses Node 22, ML service uses Python 3.11.

## Dependencies to Know

- **@nestjs/\***: Framework and ecosystem
- **@prisma/client**: ORM for database access
- **class-validator/transformer**: DTO validation
- **passport**: Authentication strategies
- **@langchain/\***: LLM integrations (Gemini, Groq, Ollama)
- **bullmq**: Job queue for async tasks
- **zod**: Frontend form validation
- **next**: React framework with SSR
- **recharts**: Frontend charting library

## Common Gotchas

1. **Prisma Client**: Run `npx prisma generate` after pulling schema changes
2. **Environment Variables**: Frontend needs `NEXT_PUBLIC_` prefix to be accessible in browser
3. **Next.js Rewrites**: API routes rewrite to `http://localhost:3003/api/v1` (NOT 4000)
4. **Python Virtual Environment**: Always activate venv before running ML service commands
5. **npm ci vs npm install**: CI uses `npm ci --legacy-peer-deps` to lock versions
6. **Legacy Peer Deps**: Some dependencies require this flag due to peer dependency conflicts

## Performance Notes

- **Turbopack** in dev mode speeds up frontend builds (15+ seconds saved)
- **NestJS incremental compilation**: `tsconfig.json` has `"incremental": true`
- **ML Service timeout**: Configured to 2.5s; adjust `PY_ML_TIMEOUT_MS` if needed
- **Cache busting**: Ensure `.env` files are NOT committed (use `.env.example`)

## Troubleshooting

- **Port conflicts**: If services won't start, check `lsof -i :PORT` (Linux/Mac) or `netstat -ano | findstr :PORT` (Windows)
- **Prisma migrations failing**: Run `npx prisma db push` to sync schema with database
- **Python import errors**: Ensure `venv` is activated and `requirements.txt` is installed
- **Node modules conflicts**: Run `npm ci --legacy-peer-deps` to reinstall with locked versions

---
> Source: [amank9115/AutiSense-AI](https://github.com/amank9115/AutiSense-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-08 -->
