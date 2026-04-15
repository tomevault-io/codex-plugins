# CLAUDE.md - Loyalty Platform Development Guide

This document provides comprehensive guidance for Claude AI and developers working with the Multi-Business Loyalty Platform.

## Project Overview

**Multi-Business Loyalty Platform MVP** - A platform where customers earn and redeem points across different businesses using their phone number as the primary identifier. The system includes an MCP (Model Context Protocol) implementation with AI-powered chat assistance and referral capabilities.

### Key Features
- Business registration and authentication (JWT-based)
- Multi-type loyalty programs (points, tiered, paid, referral, cashback)
- Customer points management and transaction tracking
- LLM-powered chat assistant with context awareness
- Referral system with automatic code generation
- AI-driven personalized recommendations
- Admin interface for business management
- Modern customer portal with responsive design

### Tech Stack
- **Backend**: FastAPI (Python 3) with SQLAlchemy ORM
- **Database**: SQLite (file-based, `loyalty.db`)
- **Admin UI**: Angular 16 with Tailwind CSS, PostCSS, TypeScript
- **Customer UI**: Node.js Express server with vanilla HTML/CSS/JavaScript
- **Auth**: JWT (python-jose) with bcrypt password hashing
- **AI Integration**: Google Gemini API (optional), MCP protocol for context handling
- **Build Tools**: npm, pip, Angular CLI

## Directory Structure

```
/Loyalty
├── backend/                           # FastAPI backend
│   ├── main.py                       # FastAPI app initialization & routing
│   ├── models.py                     # SQLAlchemy ORM models
│   ├── schemas.py                    # Pydantic request/response schemas
│   ├── database.py                   # SQLite connection setup
│   ├── auth.py                       # JWT auth & password hashing
│   ├── crud.py                       # Basic CRUD operations
│   ├── crud_loyalty_programs.py      # Loyalty program logic
│   ├── requirements.txt              # Python dependencies
│   ├── loyalty.db                    # SQLite database
│   ├── routers/
│   │   ├── auth.py                   # /auth endpoints
│   │   ├── loyalty.py                # /customers & /transactions endpoints
│   │   ├── loyalty_programs.py       # /loyalty-programs endpoints
│   │   ├── extra.py                  # Recommendations & referrals
│   │   ├── mcp_server.py             # MCP tool endpoints
│   │   └── __pycache__/
│   ├── migrate.py                    # Database migration scripts
│   ├── migrate_referral.py
│   ├── migrate_referral_code.py
│   ├── populate_econet_demo.py       # Demo data generator
│   └── __pycache__/
├── frontend/                          # Angular admin UI
│   ├── src/
│   │   ├── app/
│   │   │   ├── app.module.ts         # Main module
│   │   │   ├── app-routing.module.ts # Routes
│   │   │   ├── app.component.ts      # Root component
│   │   │   ├── components/           # UI components
│   │   │   ├── services/             # HTTP services
│   │   │   ├── guards/               # Route guards
│   │   │   ├── interceptors/         # HTTP interceptors
│   │   │   └── models/               # TypeScript interfaces
│   │   ├── assets/                   # Static assets
│   │   ├── environments/             # Environment configs
│   │   └── main.ts
│   ├── angular.json                  # Angular build config
│   ├── tsconfig.json                 # TypeScript config
│   ├── package.json                  # npm dependencies
│   ├── postcss.config.js             # PostCSS/Tailwind config
│   └── tailwind.config.js
├── customer-ui/                       # Express + vanilla JS customer portal
│   ├── server-new.js                 # Express server (main entry)
│   ├── app.js                        # Client-side app logic
│   ├── index.html                    # Customer portal UI
│   ├── gemini-assistant.js           # Gemini AI integration
│   ├── package.json
│   ├── .env.example                  # Environment variables template
│   ├── .env                          # Actual env vars (don't commit!)
│   ├── GEMINI_SETUP.md               # Gemini AI setup guide
│   ├── IMPLEMENTATION_SUMMARY.md
│   └── test_mcp_integration.js
├── instructions.md                    # Original project requirements
├── README.md                          # User-facing documentation
├── MCP_README.md                      # MCP architecture documentation
├── REFERRAL_SYSTEM_GUIDE.md           # Referral system documentation
├── quick-start.sh                     # One-command startup script
├── start_mcp.sh                       # MCP-specific startup script
├── test_*.py                          # Python test files
├── test_mcp_server.py                # MCP server tests
├── test_referral_system.py           # Referral system tests
├── test_mcp_integration.js           # JavaScript MCP tests
└── logs/                             # Application logs directory
```

## Architecture Overview

### MCP Implementation
The system uses the Model Context Protocol to maintain conversation context:
- **MCP Client**: `customer-ui/app.js` - Sends context-rich requests to MCP server
- **MCP Server**: `backend/routers/mcp_server.py` - Executes tools with context handling
- **Context Management**: Session ID + phone number + user state preserved across requests

### API Architecture
```
Frontend/Client
    ↓
Express.js (customer-ui/server-new.js)
    ↓
FastAPI Backend (backend/main.py)
    ├── /auth/* (JWT authentication)
    ├── /customers/* (customer endpoints)
    ├── /transactions/* (point management)
    ├── /loyalty-programs/* (program management)
    ├── /mcp/* (MCP tools)
    └── SQLite Database
```

## Database Schema

### Core Models
- **Business**: email (unique), name, contact_person, password_hash, loyalty_rate, created_at
- **Customer**: phone_number (unique), total_points, referral_code, created_at
- **Transaction**: business_id, customer_id, amount_spent, points_earned, timestamp, type (enum)
- **LoyaltyProgram**: business_id, name, program_type, earn_rate, active, created_at
- **CustomerMembership**: customer_id, loyalty_program_id, points, is_paid_member, current_tier_id
- **Reward**: loyalty_program_id, name, points_required, is_active, stock_limit
- **TierLevel**: loyalty_program_id, name, min_points, benefits, multiplier
- **Referral**: referrer_id, referred_id, business_id, points_awarded, created_at

### Enums
- **TransactionType**: EARN, REDEMPTION, REFERRAL, CASHBACK
- **LoyaltyProgramType**: POINTS, TIERED, PAID, REFERRAL, CASHBACK

## API Endpoints

### Authentication (`/auth`)
- `POST /auth/register_business` - Register new business
- `POST /auth/login_business` - Business login (returns JWT token)
- `GET /auth/me` - Get current business (protected)

### Loyalty Management (`/transactions` & `/customers`)
- `POST /transactions/add_points` - Add points for customer (protected)
- `POST /transactions/redeem_points` - Redeem points (protected)
- `GET /customers/points/{phone_number}` - Get customer points (public)
- `POST /customers/redeem-reward` - Redeem reward (public)
- `GET /customers/available-rewards/{phone_number}` - View available rewards (public)

### Loyalty Programs (`/loyalty-programs`)
- `POST /loyalty-programs/` - Create loyalty program (protected)
- `GET /loyalty-programs/business/{business_id}` - Get business programs (protected)
- `GET /loyalty-programs/{program_id}` - Get program details (public)
- `PUT /loyalty-programs/{program_id}` - Update program (protected)

### MCP Tools (`/mcp`)
- `POST /mcp/tool` - Execute MCP tool
  - Available tools: `check_points`, `get_recommendations`, `explain_referrals`, `get_business_info`, `get_customer_transactions`, `get_analytics`

### Recommendations & Referrals
- `GET /customers/recommendations/{phone_number}` - Get personalized recommendations
- `POST /referrals/track` - Track referral
- `GET /customers/{phone_number}/referral-code` - Get customer's referral code

## Development Workflows

### Backend Development

#### Setup
```bash
cd backend
python3 -m venv venv
source venv/bin/activate  # or `venv\Scripts\activate` on Windows
pip install -r requirements.txt
```

#### Running
```bash
uvicorn main:app --reload --host 0.0.0.0 --port 8000
# API docs: http://localhost:8000/docs (Swagger UI)
# ReDoc: http://localhost:8000/redoc
```

#### Key Files
- **main.py**: FastAPI app setup, CORS config, router registration
- **models.py**: SQLAlchemy model definitions - modify to change schema
- **schemas.py**: Pydantic validation schemas - used by all endpoints
- **database.py**: SQLite connection - currently uses `./loyalty.db`

#### Adding New Endpoints
1. Create new router file in `routers/`
2. Define Pydantic schemas in `schemas.py`
3. Add models to `models.py` if needed
4. Register router in `main.py` with `app.include_router()`
5. Document endpoint and add to MCP tools if needed

#### Database Migrations
- Use `migrate.py` scripts for schema changes
- ORM handles table creation via `models.Base.metadata.create_all(bind=engine)`
- For complex migrations, create new migration script following existing patterns

### Frontend Development (Angular Admin UI)

#### Setup
```bash
cd frontend
npm install
# Ensure Node 16+ (ideally 18+)
```

#### Running
```bash
npm start  # or `ng serve`
# App runs on http://localhost:4200
```

#### Key Files
- **app.module.ts**: Imports components, services, modules
- **app-routing.module.ts**: Route definitions
- **services/**: HTTP clients (REST API communication)
- **components/**: Angular components for each page
- **models/**: TypeScript interfaces matching backend schemas

#### Styling
- Uses **Tailwind CSS** for utility classes
- `tailwind.config.js` for customization
- `postcss.config.js` for build integration
- Components use `.component.css` for scoped styles or inline Tailwind classes in templates

#### Building
```bash
npm run build
# Output in dist/loyalty-admin/
```

### Customer Portal Development

#### Setup
```bash
cd customer-ui
npm install
# Or use Python: python3 -m http.server 5500
```

#### Running
```bash
npm start  # Runs server-new.js on port 5500
# Or: node server-new.js
# Customer UI: http://localhost:5500
```

#### Key Files
- **server-new.js**: Express server with Gemini AI integration
- **app.js**: Core client-side logic, MCP client implementation
- **index.html**: Customer portal markup
- **gemini-assistant.js**: Gemini API wrapper and LLM logic

#### Environment Variables
Create `.env` file:
```
GEMINI_API_KEY=your_gemini_api_key_here
BACKEND_API_URL=http://localhost:8000
PORT=5500
```

## Testing

### Backend Tests
```bash
# Test MCP server implementation
python test_mcp_server.py

# Test referral system
python test_referral_system.py

# Test with curl
curl -X POST http://localhost:8000/mcp/tool \
  -H "Content-Type: application/json" \
  -d '{"tool": "check_points", "parameters": {"phone_number": "+1234567890"}, "context": {}}'
```

### Frontend Tests (Angular)
```bash
# Unit tests
npm test

# E2E tests (if configured)
ng e2e
```

### Integration Testing
```bash
# JavaScript MCP integration tests
node test_mcp_integration.js

# Full stack test
./quick-start.sh
```

## Authentication & Authorization

### JWT Implementation
- **Secret Key**: `auth.SECRET_KEY` (currently hardcoded - should use env var in production)
- **Algorithm**: HS256
- **Token Expiry**: 30 minutes (configurable in `auth.py`)
- **Storage**: JWT in Authorization header: `Bearer {token}`

### Protected Endpoints
- Require `Authorization: Bearer {token}` header
- Token validated via `get_current_business()` dependency
- Uses `oauth2_scheme` from `auth.py`

### No Authentication
- Public customer endpoints (check points, view rewards, etc.)
- For production: implement OTP or PIN for customer endpoints

## Key Architectural Decisions

### Phone Number as Identifier
- Unique identifier for customers (enforced via DB constraint)
- Format: 9-15 digits, with optional `+` prefix (validated in routes)
- Enables SMS-based interaction

### Loyalty Program Types
- Flexible schema to support multiple program models
- Each business can have multiple programs
- Customers have memberships per program (points per program)
- Backward compatible with legacy `total_points` field

### MCP Protocol
- Separates concerns between UI and business logic
- Context-aware tool execution
- Enables multi-client support
- Future-ready for LLM integration

### SQLite for MVP
- Fast development cycle
- File-based, easy deployment
- Sufficient for MVP scale
- Migrate to PostgreSQL for production

## Code Conventions

### Python Backend
- **File naming**: `snake_case.py`
- **Function/variable naming**: `snake_case`
- **Class naming**: `PascalCase`
- **Imports**: System → third-party → local (organized at top)
- **Error handling**: Raise exceptions with descriptive messages, caught at router level
- **Type hints**: Used in schemas and function signatures
- **Database queries**: Use SQLAlchemy ORM exclusively, avoid raw SQL

### TypeScript/Angular Frontend
- **File naming**: `kebab-case.component.ts`, `kebab-case.service.ts`
- **Class naming**: `PascalCase`
- **Variable naming**: `camelCase`
- **Type annotations**: Required for all public methods
- **Angular patterns**: Components, services, guards, interceptors
- **Tailwind classes**: Used extensively for styling

### JavaScript (Customer UI)
- **File naming**: `camelCase.js` or `kebab-case.js`
- **Class naming**: `PascalCase`
- **Function naming**: `camelCase`
- **Modern ES6+**: Arrow functions, const/let (no var)
- **Comments**: JSDoc style for complex functions

## Common Tasks

### Adding a New Loyalty Program Type
1. Add new enum value in `models.LoyaltyProgramType`
2. Create new schema in `schemas.py` (e.g., `NewProgramCreate`)
3. Add logic in `crud_loyalty_programs.py`
4. Update router in `loyalty_programs.py` if needed

### Adding a New MCP Tool
1. Define tool logic as async function in `mcp_server.py`
2. Add to tool dispatch in `execute_mcp_tool()`
3. Document in `MCP_README.md`
4. Test with `test_mcp_server.py`

### Implementing a New Endpoint
1. Add Pydantic schema in `schemas.py`
2. Add model/update logic in appropriate router
3. Register in `main.py` if new router
4. Test with Swagger UI or curl
5. Document in README

### Debugging
- **Backend**: Check logs, use `print()` statements, enable FastAPI debug mode
- **Frontend**: Browser DevTools, Angular debug extension
- **Customer UI**: Check server logs and browser console
- **Database**: Use SQLite CLI or DB browser tools

## Configuration

### Backend Configuration
- **Database**: `database.py` - `SQLALCHEMY_DATABASE_URL`
- **JWT Secret**: `auth.py` - `SECRET_KEY` (use env var in production!)
- **CORS**: `main.py` - currently allows all origins
- **Port**: 8000 (configurable via uvicorn CLI)

### Frontend Configuration
- **API Base URL**: Usually `http://localhost:8000` (set in services)
- **Environment files**: `environments/environment.ts` and `environment.prod.ts`

### Customer UI Configuration
- **Gemini API Key**: `.env` file - `GEMINI_API_KEY`
- **Backend URL**: `.env` file - `BACKEND_API_URL`
- **Port**: `.env` file - `PORT` (default: 5500)

## Performance Considerations

### Database
- Query optimization: Limit result sets, use indexed fields (phone_number, email)
- Transactions: Atomic operations for points/redemption
- Scaling: Switch to PostgreSQL for production

### API Response Times
- Cache business/program data if changed infrequently
- Implement pagination for large result sets
- Use async operations for I/O

### Frontend
- Lazy load components in Angular
- Optimize bundle size with tree-shaking
- Use OnPush change detection where appropriate

## Security Notes

### Current Limitations
- JWT secret hardcoded (use env vars)
- CORS allows all origins (restrict in production)
- No rate limiting
- Phone numbers passed in URLs (consider POST)
- No input sanitization for special characters

### Recommendations
- Use `.env` file for all secrets
- Implement rate limiting (e.g., Redis-based)
- Add request validation/sanitization
- Use HTTPS only in production
- Implement API key rotation
- Add audit logging for sensitive operations
- Implement CSRF protection for web forms

## Deployment

### Quick Start
```bash
./quick-start.sh
# Or with MCP-specific setup:
./start_mcp.sh
```

### Production Checklist
- [ ] Change JWT secret to strong random value
- [ ] Set CORS to specific origins
- [ ] Use environment variables for all secrets
- [ ] Enable HTTPS/TLS
- [ ] Set up database backups
- [ ] Implement rate limiting
- [ ] Enable GZIP compression
- [ ] Set up error monitoring (Sentry, etc.)
- [ ] Implement proper logging
- [ ] Use production database (PostgreSQL)
- [ ] Enable caching (Redis)
- [ ] Set up CI/CD pipeline
- [ ] Test disaster recovery

## Troubleshooting

### Backend Won't Start
- Check Python version (3.8+)
- Verify dependencies: `pip install -r requirements.txt`
- Check port 8000 isn't in use: `lsof -i :8000`
- Delete `__pycache__` and try again

### Frontend Won't Start
- Check Node version (16+): `node --version`
- Clear node_modules: `rm -rf node_modules && npm install`
- Check port 4200 isn't in use: `lsof -i :4200`

### Customer UI Connection Issues
- Verify backend running on port 8000
- Check `.env` file `BACKEND_API_URL`
- Check browser console for CORS errors
- Verify Gemini API key if using AI features

### Database Issues
- Delete `loyalty.db` to reset (loses all data)
- Run migrations if schema changed
- Check file permissions on `loyalty.db`

## References

- **FastAPI**: https://fastapi.tiangolo.com/
- **SQLAlchemy**: https://docs.sqlalchemy.org/
- **Angular**: https://angular.io/
- **Tailwind CSS**: https://tailwindcss.com/
- **MCP Protocol**: See `MCP_README.md`
- **Referral System**: See `REFERRAL_SYSTEM_GUIDE.md`

## Contact & Support

For questions, check:
1. `instructions.md` - Original requirements
2. `MCP_README.md` - MCP architecture details
3. `REFERRAL_SYSTEM_GUIDE.md` - Referral/recommendation details
4. Backend logs in `/logs/` directory
5. Browser DevTools for frontend issues

## Future Roadmap

### Phase 3 (Planned)
- Real SMS integration (Twilio)
- Push notifications
- Mobile app
- Advanced analytics dashboard
- Machine learning recommendations
- Social media integration
- Payment processing for paid memberships

### Optimization Tasks
- Implement caching layer
- Add full-text search for businesses
- Optimize database queries
- Implement pagination everywhere
- Add WebSocket for real-time updates

### Security Enhancements
- OAuth2 social login
- Two-factor authentication
- End-to-end encryption
- Audit logging
- PCI DSS compliance for payments

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/digiland)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/digiland)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
