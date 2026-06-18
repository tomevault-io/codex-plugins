# Project Overview: SCHEMABRIDGE - Enterprise XML Transformation Platform

**Production-ready XML mapping, transformation, and AI-powered invoice extraction system with enterprise security.**

## A. Current Development Setup and Tech Stack

### 🏗️ Architecture Overview
**Monorepo** with Frontend, Backend, ML Service, WebSocket Server, and Background Workers.

### **Frontend (Directory: `frontend/`)**
- **Stack:** React 19 + Vite 7
- **Dev Environment:** Port 5173, HMR enabled
- **API Proxy:** All `/api` requests → `http://localhost:3000`
- **Key Features:**
  - Visual XML mapping editor with drag-and-drop
  - Interactive schema tree viewer
  - AI mapping suggestions UI (75-90% accuracy)
  - Admin dashboard (user management, RBAC, security monitoring)
  - PDF invoice viewer with real-time extraction
  - Glassmorphic UI with auto-logout (10min idle)
- **Dependencies:** React Router, Socket.io-client, Chart.js, jsPDF, react-pdf, TanStack Table

### **Backend (Directory: `backend/`)**
- **Stack:** AWS SAM (Lambda + API Gateway) + Node.js 18
- **Dev Environment:** Port 3000 (SAM Local)
- **Database:** PostgreSQL 13 (Docker, port 5432)
- **Core Services:**
  - **XML Transformation** (`/api/transform`, `/api/webhook/transform`)
  - **Schema Parsing** (`/api/schema/parse`)
  - **AI Mapping** (`/api/ai/suggest-mapping`, `/api/ai/batch-suggest`)
  - **Invoice Extraction** (`/api/invoice/*` - OCR + LayoutLMv3 + Gemini)
  - **Rossum Integration** (`/api/webhook/rossum`)
  - **Admin APIs** (`/api/admin/*` - users, roles, security audit)
  - **Security** - RBAC, JWT auth, XML security validation, audit logging
- **Key Services:**
  - `xmlParser.service.js` - **PROTECTED** XML parsing & transformation logic
  - `aiMapping.service.js` - AI-powered field mapping (Gemini 2.0 Flash)
  - `invoiceExtraction.service.js` - ML-based invoice data extraction
  - `user.service.js` - User management with RBAC
  - `selfLearning.service.js` - Self-learning from user corrections
- **Dependencies:** Express, @xmldom/xmldom, pg (PostgreSQL), jsonwebtoken, bcrypt, @google/generative-ai, Bull (job queue), Socket.io, Redis, Sharp, Multer

### **ML Service (Directory: `services/`)**
- **Stack:** Python 3.10 + Flask
- **Architecture (Ultra-Lightweight <6GB):** 
  - **P1: OCR Service** - PaddleOCR + PP-Structure (~500MB)
  - **P2: Extractor Service** - GLiNER small model (~300MB)
  - **P3: API Gateway** - FastAPI + Label Studio HITL integration (~100MB)
- **Dev Ports:** 5002 (OCR), 5003 (Extractor), 8000 (Gateway)
- **Purpose:** CPU-only invoice extraction with spatial context augmentation
- **HITL:** Automatic routing to Label Studio when confidence < 0.90
- **Proven Architecture:** See `.github/extraction_arch.md`

### **WebSocket Server (File: `backend/socketServer.js`)**
- **Stack:** Socket.io + Node.js
- **Dev Environment:** Port 3001
- **Purpose:** Real-time updates for invoice extraction jobs
- **Features:** Job progress, extraction status, live error reporting

### **Background Workers (Directory: `backend/workers/`)**
- **Stack:** Bull (Redis-backed job queue)
- **Purpose:** Asynchronous invoice processing
- **Worker:** `extractionWorker.js` - Handles ML inference jobs

### **Database (PostgreSQL 13)**
- **Container:** Docker Compose, port 5432
- **Schema:** `backend/db/init.sql` + migrations (`backend/db/migrations/`)
- **Key Tables:**
  - `users`, `organizations`, `roles`, `permissions` (RBAC)
  - `mappings`, `schemas` (XML transformation)
  - `invoices`, `line_items`, `invoice_audit_log` (invoice extraction)
  - `security_audit_log` (security events)
  - `api_keys`, `webhooks` (API management)
- **Features:** Row-Level Security (RLS), audit logging, multi-tenancy

### **API Endpoints Overview**

#### Core Microservices (Ultra-Lightweight IDP)
- **P1 OCR Service:** `POST http://localhost:5002/process-document` - PaddleOCR + spatial context
- **P2 Extractor Service:** `POST http://localhost:5003/extract-customs-fields` - GLiNER NER (~300MB)
- **P3 API Gateway:** `POST http://localhost:8000/api/v1/invoice/upload` - HITL orchestration
- **Label Studio:** `http://localhost:8080` - Human-in-the-Loop corrections

#### Legacy Transformation & Mapping
- `POST /api/transform` - Synchronous XML transformation
- `POST /api/webhook/transform` - Async webhook transformation
- `POST /api/schema/parse` - Parse XML to tree structure
- `POST /api/ai/suggest-mapping` - AI field mapping (single)
- `POST /api/ai/batch-suggest` - AI field mapping (batch)

#### Invoice Extraction (AI/ML)
- `POST /api/invoice/upload` - Upload invoice (PDF/image)
- `GET /api/invoice/:id` - Get extraction results
- `POST /api/invoice/:id/correct` - Submit user corrections (self-learning)
- `GET /api/invoice/:id/audit` - Get audit trail
- `GET /api/analytics/accuracy` - ML accuracy metrics

#### Rossum Integration
- `POST /api/webhook/rossum` - Rossum webhook receiver
- Converts Rossum JSON → XML → Transformation → Destination webhook

#### Admin & Security
- `POST /api/auth/login` - JWT authentication
- `GET /api/admin/users` - User management (Admin only)
- `POST /api/admin/users` - Create user
- `PUT /api/admin/users/:id` - Update user/roles
- `DELETE /api/admin/users/:id` - Delete user
- `GET /api/admin/security/audit` - Security audit log
- `GET /api/admin/analytics/failed-auth` - Failed login attempts

### **Development Workflow**

#### Start Microservices (Ultra-Lightweight)
```bash
bash setup-idp-microservices.sh  # Complete setup: PaddleOCR + GLiNER + HITL
```

**Services available:**
- P1 OCR Service: http://localhost:5002 (PaddleOCR)
- P2 Extractor Service: http://localhost:5003 (GLiNER)
- P3 API Gateway: http://localhost:8000 (HITL orchestration)
- Label Studio: http://localhost:8080 (admin@localhost / admin123)

#### Start Legacy System
```bash
bash start-dev.sh  # Database + Backend + Frontend + Socket + Worker
```

#### Individual Services
```bash
bash start-db.sh              # PostgreSQL only
bash start-backend.sh         # AWS SAM Local (port 3000)
bash start-frontend.sh        # Vite dev server (port 5173)
bash start-socket-server.sh   # Socket.io (port 3001)
bash start-worker.sh          # Bull worker (extraction jobs)
docker-compose up -d ocr-service extractor-service api-gateway  # Microservices only
```

### **Testing**
- Microservices pipeline: `tests/test-microservices-pipeline.sh`
- Integration tests: `tests/test-integration.sh`
- Security tests: `tests/test-security.sh`
- Admin API tests: `tests/test-admin-api.sh`
- Rossum webhook tests: `tests/test-rossum-webhook.sh`

### **Documentation**
- **Microservices:** `docs/microservices/MICROSERVICES_IMPLEMENTATION.md`
- **Architecture:** `.github/extraction_arch.md`
- **Analysis:** `.github/extraction_refactor_analysis.md`
- **Setup:** `docs/setup/SETUP.md`
- **API Reference:** `docs/api/API_DOCUMENTATION.md`
- **Security:** `docs/security/` (ISO 27001 - 70% compliance)
- **Rossum Integration:** `docs/rossum/ROSSUM_DOCS_INDEX.md`
- **Admin Guide:** `docs/admin/ADMIN_PANEL_GUIDE.md`

---

## B. 🛑 ABSOLUTE CONSTRAINTS: DO NOT MODIFY 🛑

The existing XML parsing, transformation, and selector logic is considered **stable and production-ready**. **DO NOT modify this code** unless explicitly directed by the user for a new feature outside of core parsing mechanics.

### **Protected Files & Logic:**

1. **`backend/services/xmlParser.service.js`**
   - XML parsing functions (`parseXmlToTree`, `convertTreeToXml`)
   - XPath selector logic
   - Tree structure building
   - Transformation engine

2. **Related XML Processing:**
   - Any file with XML tree creation logic
   - Mapping/transformation rule execution
   - XPath evaluation functions

**Exception:** User explicitly requests new XML features (e.g., new transformation rules, schema validation)

## C. 🎯 Copilot Behavior Rules (Accuracy & Safety) 🎯

### 1. **Code Safety and Minimal Intervention**
- **Rule:** Prioritize minimal viable changes. Only generate the exact code needed for the user's request.
- **Rule:** **Never** add unnecessary comments, redundant code, or introduce external libraries/dependencies unless explicitly told to install them.
- **Rule:** Before any change, confirm that the request does not violate the Absolute Constraints (Section B).
- **Rule:** Do not modify working production code unless specifically requested.

### 2. **Context Window Management**
- **Rule:** When at **95% of context window**, Copilot **must** summarize context using last 5 prompts, output the following message, and wait for user confirmation before proceeding. Once user confirms, continue the conversation picking up from the last prompt using summarized context:

> **[Context Refresh Required]**  
> I am at 95% context capacity. Summarizing the last 5 prompts and resetting internal context while preserving:
> - Core project constraints (Section B)
> - Current task progress
> - Active conversation thread
> 
> Please confirm to continue with refreshed context.

### 3. **Development Best Practices**
- **Rule:** Always test changes when possible (use available test scripts)
- **Rule:** Follow existing code patterns and conventions
- **Rule:** Respect the monorepo structure (frontend/, backend/, docs/, scripts/, tests/)
- **Rule:** Use environment variables for sensitive data (JWT_SECRET, GEMINI_API_KEY, etc.)
- **Rule:** Maintain security standards (RBAC, audit logging, XML validation)

### 4. **File Organization**
- **Rule:** Place setup scripts in `scripts/setup/`
- **Rule:** Place test scripts in `tests/`
- **Rule:** Place documentation in `docs/` with appropriate subdirectory
- **Rule:** Keep root directory clean (only start-*.sh scripts)

### 5. **Database Changes**
- **Rule:** Create new migrations in `backend/db/migrations/` with sequential numbering
- **Rule:** Never modify `init.sql` directly (use migrations)
- **Rule:** Test migrations with `backend/db/run-migrations.sh`

### 6. **Security & Compliance**
- **Rule:** All new endpoints must include authentication/authorization
- **Rule:** Log security events to `security_audit_log` table
- **Rule:** Validate and sanitize XML input (XXE, XSS prevention)
- **Rule:** Use RBAC permissions for resource access

---

## D. 📝 VS Code Settings (Slowing Inline Suggestions)

To reduce aggressive inline suggestions and improve goal understanding, update your VS Code settings with a slight delay before suggestions appear.

**File:** `.vscode/settings.json`

Merge the following settings into your existing configuration:

```json
{
    "python-envs.defaultEnvManager": "ms-python.python:system",
    "python-envs.pythonProjects": [],
    
    // --- Copilot / Editor Settings for Careful Interaction ---
    
    "editor.inlineSuggest.minShowDelay": 500, 
    // ^ Delay in milliseconds before inline suggestions appear. Set to 500ms (0.5 seconds) to slow down aggressive suggestions.

    "github.copilot.nextEditSuggestions.enabled": false,
    // ^ Disabling this prevents Copilot from trying to predict and offer multi-line refactoring/next steps, forcing single-file focus.

    "github.copilot.internal.editor.inlineSuggest.debounce": 200,
    // ^ Adds a slight debounce to when completions are requested, improving accuracy.

    "github.copilot.nextEditSuggestions.fixes": false
    // ^ Disables automatically suggesting fixes based on diagnostics, ensuring you control the fix.
}
```

**Purpose:**
- `minShowDelay: 500` - Adds 0.5s pause before suggestions appear
- `nextEditSuggestions.enabled: false` - Prevents multi-line prediction spirals
- `debounce: 200` - Improves completion accuracy
- `nextEditSuggestions.fixes: false` - User controls all fixes

---

## E. 🚀 Quick Reference

### **Common Tasks**

```bash
# First-time setup
bash scripts/setup/setup-project.sh

# Daily development
bash start-dev.sh

# Create new migration
touch backend/db/migrations/011_your_feature.sql

# Run migrations
bash backend/db/run-migrations.sh

# Test transformation API
curl -X POST http://localhost:3000/api/transform \
  -H "Content-Type: application/json" \
  -d '{"sourceXml":"<Invoice><Amount>100</Amount></Invoice>","mapping":[{"source":"Invoice/Amount","destination":"Payment/Total"}]}'

# View logs
tail -f backend/extraction-worker.log
tail -f backend/socket-server.log
```

### **Default Credentials**
- **Admin:** `d.radionovs@gmail.com` / `password123`
- **Dev User:** `dev@example.com` / `password123`

### **Service URLs**
- Frontend: http://localhost:5173
- Backend API: http://localhost:3000
- Socket.io: http://localhost:3001
- ML Service: http://localhost:5001 (when running)
- PostgreSQL: localhost:5432

### **Key Directories**
- `/backend/services/` - Business logic
- `/backend/routes/` - API endpoints
- `/backend/middleware/` - Auth, RBAC, security
- `/frontend/src/pages/` - React pages
- `/frontend/src/components/` - UI components
- `/docs/` - All documentation
- `/scripts/` - Utility scripts
- `/tests/` - Test scripts

---

## F. 📊 Project Status

### **Completed Features** ✅
- XML transformation engine (production-ready)
- Visual mapping editor
- AI field mapping (75-90% confidence)
- Invoice extraction (90-95% accuracy with Gemini)
- Admin dashboard (user management, RBAC)
- Security implementation (ISO 27001 - 70% compliant)
- Rossum integration (95% complete)
- Self-learning from corrections
- Real-time job updates (Socket.io)

### **Known Limitations**
- Rossum XML export endpoint (5% remaining)
- Rate limiting (planned)
- Data encryption at rest (planned)

### **Production Ready**
- Core XML transformation ✅
- Security & RBAC ✅
- Admin dashboard ✅
- Invoice extraction ✅
- API documentation ✅

---

---
> Source: [Dwys97/ROSSUMXML](https://github.com/Dwys97/ROSSUMXML) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-18 -->
