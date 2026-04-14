# Biomedical Analyst Roleplay Platform

## Project Overview

An educational simulation platform where students roleplay as biomedical data analysts, interacting with LLM-powered "Stakeholders" based on real datasets. Uses a sophisticated **meta-prompt architecture** where an Architect LLM generates custom scenarios and Actor system prompts for dynamic, dataset-aware conversations.

## Architecture

### Tech Stack
- **Backend**: Python Flask API (serverless-ready for Vercel)
- **Frontend**: React + Vite SPA
- **Database**: Supabase (PostgreSQL)
- **LLM**: Google Gemini (gemini-1.5-flash)
- **Deployment**: Vercel (monolithic)

### Meta-Prompt Architecture

**Two-Stage LLM System:**

1. **Architect LLM** (Stage 1): Generates comprehensive scenarios
   - Input: Dataset metadata (columns, sample data, quality notes)
   - Output: Structured JSON with scenario details + custom Actor system prompt
   - Purpose: Creates unique, educational scenarios tailored to each dataset

2. **Actor LLM** (Stage 2): Roleplays as stakeholder
   - Input: Architect-generated system prompt + chat history
   - Output: In-character responses that guide students
   - Purpose: Provides business context, refuses to write code, teaches data cleaning

### Key Innovation

Instead of hardcoded chat prompts, the Architect LLM **generates custom system prompts** for the Actor that:
- Know specific column names from the dataset
- Mention data quality issues naturally
- Guide students toward data cleaning challenges
- Stay in character and refuse to write code

## Project Structure

```
tubes-pdb/
├── api/
│   └── index.py                 # Vercel serverless entry point
├── backend/
│   ├── app.py                   # Flask app factory
│   ├── config.py                # Environment configuration
│   ├── models/                  # Pydantic data models
│   │   ├── user.py             # Lecturer model
│   │   ├── student.py          # Student model
│   │   ├── dataset.py          # Dataset with meta-prompt fields
│   │   ├── assignment.py       # Scenario model
│   │   ├── chat_message.py     # Chat history
│   │   ├── submission.py       # Student submissions
│   │   └── grade.py            # Grading
│   ├── routes/                  # API endpoints (blueprints)
│   │   ├── auth.py             # Authentication
│   │   ├── datasets.py         # Dataset CRUD
│   │   ├── assignments.py      # JIT scenario generation
│   │   ├── chat.py             # Chat with Actor LLM
│   │   ├── submissions.py      # Submission handling
│   │   └── grading.py          # Grading interface
│   ├── services/                # Business logic
│   │   ├── llm_service.py      # Gemini integration (Architect + Actor)
│   │   ├── auth_service.py     # JWT + bcrypt
│   │   └── assignment_service.py # JIT generation logic
│   └── utils/
│       ├── db.py               # Supabase client
│       └── validators.py       # Input validation
├── frontend/
│   └── src/
│       ├── pages/
│       │   ├── student/        # Student UI
│       │   └── lecturer/       # Lecturer UI
│       ├── context/            # React context (auth)
│       ├── services/           # API client (axios)
│       └── index.css           # Design system
├── docs/
│   ├── prd.md                  # Product requirements
│   ├── meta_prompt.md          # Meta-prompt architecture
│   ├── example_dataset.md      # Dataset examples
│   ├── db_schema.sql           # Database schema
│   └── vercel_deployment.md    # Deployment guide
├── requirements.txt            # Python dependencies
├── vercel.json                 # Vercel config (monolithic)
└── .env.example                # Environment template
```

## Database Schema

### Tables

**users** (Lecturers)
- `id` (UUID, PK)
- `email` (unique)
- `password_hash` (bcrypt)
- `created_at`

**students**
- `nim` (VARCHAR, PK) - Student ID
- `name`
- `created_at`

**datasets** (Enhanced for meta-prompt)
- `id` (UUID, PK)
- `name`
- `url` (Google Drive link)
- `metadata_summary`
- `columns_list` (TEXT[]) - Column names
- `sample_data` (TEXT) - CSV format
- `data_quality_notes` (TEXT) - Known issues
- `created_at`

**assignments**
- `id` (UUID, PK)
- `student_nim` (FK → students, unique)
- `dataset_id` (FK → datasets)
- `scenario_json` (JSONB) - Architect output
- `created_at`

**chat_messages**
- `id` (UUID, PK)
- `assignment_id` (FK → assignments)
- `sender` ('student' | 'ai')
- `content` (TEXT)
- `timestamp`

**submissions**
- `id` (UUID, PK)
- `assignment_id` (FK → assignments)
- `link_url` (Google Drive/Colab)
- `submission_type` ('progress' | 'final')
- `created_at`

**grades**
- `assignment_id` (UUID, PK, FK → assignments)
- `score` (0-100)
- `feedback` (TEXT)
- `created_at`

## API Endpoints

### Authentication
- `POST /api/auth/student/login` - NIM-only login
- `POST /api/auth/lecturer/login` - Email/password
- `POST /api/auth/lecturer/register` - Create lecturer
- `POST /api/auth/students/upload-roster` - Bulk upload (CSV)

### Datasets
- `GET /api/datasets` - List all (lecturer only)
- `POST /api/datasets` - Create with meta-prompt fields
- `DELETE /api/datasets/:id` - Delete

### Assignments
- `GET /api/assignments/me` - Get/create (JIT generation)
- `POST /api/assignments/regenerate` - Delete & regenerate

### Chat
- `GET /api/chat/:assignment_id/messages` - Get history
- `POST /api/chat/:assignment_id/message` - Send & get AI response

### Submissions
- `GET /api/submissions/:assignment_id` - Get submissions
- `POST /api/submissions` - Create submission

### Grading
- `GET /api/grading/students` - All students with data
- `POST /api/grading/grade` - Submit grade

## Key Flows

### Just-in-Time (JIT) Scenario Generation

1. Student logs in with NIM
2. System checks if assignment exists
3. If NO:
   - Select random dataset
   - Call Architect LLM with dataset metadata
   - Architect generates scenario + Actor system prompt
   - Save to `assignments.scenario_json`
4. Return assignment to student

### Chat Flow

1. Student sends message
2. Backend fetches assignment's `scenario_json`
3. Extract `persona_system_instruction` (generated by Architect)
4. Call Actor LLM with:
   - System: `persona_system_instruction`
   - History: Last 20 messages
   - User: New message
5. Save both messages to `chat_messages`
6. Return AI response

## Environment Variables

```bash
# Supabase
SUPABASE_URL=https://xxx.supabase.co
SUPABASE_KEY=anon-key
SUPABASE_SERVICE_KEY=service-role-key

# Google Gemini
GEMINI_API_KEY=your-key

# JWT
JWT_SECRET=generated-secret

# Optional
FRONTEND_URL=http://localhost:5173
FLASK_ENV=development
```

## Authentication

- **Students**: NIM-only (no password) - acceptable for MVP
- **Lecturers**: Email + bcrypt hashed password
- **JWT tokens**: 24-hour expiration
- **Middleware**: `@require_auth(user_type)` decorator

## Design System (Frontend)

### Colors
- Primary: Blue gradient (#667eea → #764ba2)
- Secondary: Green (success states)
- Grays: Neutral backgrounds

### Typography
- Font: Inter (UI), JetBrains Mono (code)
- Scale: xs (0.75rem) to 5xl (3rem)

### Components
- Buttons: `.btn`, `.btn-primary`, `.btn-secondary`
- Cards: `.card` with shadows
- Forms: `.form-input`, `.form-textarea`
- Badges: `.badge` with variants

## Development

### Backend
```bash
# From project root
python backend/app.py
# Runs on http://localhost:5000
```

### Frontend
```bash
cd frontend
npm run dev
# Runs on http://localhost:5173
```

### Testing
```bash
# Health check
curl http://localhost:5000/api/health

# Register lecturer
curl -X POST http://localhost:5000/api/auth/lecturer/register \
  -H "Content-Type: application/json" \
  -d '{"email":"test@test.com","password":"password123"}'
```

## Deployment (Vercel)

Monolithic deployment:
- Backend: Serverless functions (`api/index.py`)
- Frontend: Static assets (`frontend/dist`)
- Routing: `/api/*` → backend, `/*` → frontend

See `docs/vercel_deployment.md` for details.

## Important Conventions

### Import Paths
- Always use `backend.` prefix when importing
- Example: `from backend.config import get_config`
- Run from project root: `python backend/app.py`

### Scenario JSON Structure
```json
{
  "scenario_title": "The Tuesday Morning ER Bottleneck",
  "difficulty_level": "Intermediate",
  "stakeholder_name": "Dr. Budi Santoso",
  "stakeholder_role": "Head of Emergency Services",
  "email_body": "Hi there, I hope this email finds you well...",
  "key_objectives": ["Calculate wait times", "Handle missing data", ...],
  "persona_system_instruction": "You are Dr. Budi Santoso... [GENERATED BY ARCHITECT]"
}
```

### Dataset Creation
Must include meta-prompt fields:
- `columns_list`: Array of column names
- `sample_data`: CSV format (first 5 rows)
- `data_quality_notes`: Known issues for teaching

## Security Considerations

- JWT tokens in localStorage
- Bcrypt password hashing (cost factor 12)
- CORS configured for frontend origin
- Input validation with Pydantic
- Service role key for admin operations
- No sensitive data in scenario JSON sent to frontend

## Common Issues

### ModuleNotFoundError
- Run from project root, not inside `backend/`
- Use `backend.` prefix in imports

### Missing environment variables
- Copy `.env.example` to `.env`
- Fill in all required values

### LLM not generating scenarios
- Check Gemini API key
- Verify dataset has `columns_list` and `sample_data`
- Review `backend/services/llm_service.py` prompts

## Resources

- [PRD](../docs/prd.md) - Product requirements
- [Meta-Prompt Architecture](../docs/meta_prompt.md) - Two-stage LLM design
- [Example Datasets](../docs/example_dataset.md) - Dataset templates
- [Deployment Guide](../docs/vercel_deployment.md) - Vercel setup

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/IritaSee) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
