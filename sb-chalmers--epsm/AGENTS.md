# EPSM - Copilot Instructions

This repository contains the **Energy Performance Simulation Manager (EPSM)**, a containerized web application for managing building energy simulations using EnergyPlus.

## Project Overview

EPSM is a full-stack application designed to streamline the process of running EnergyPlus simulations for building performance optimization. It enables building owners, researchers, and engineers to explore and evaluate energy renovation strategies across large building stocks.

**Key Purpose:** Manage materials, constructions, and building templates; run baseline simulations; create renovation scenarios; execute batch simulations; and analyze results with interactive visualizations.

## Technology Stack

### Frontend
- **Framework:** React 18 with TypeScript 5.9
- **Build Tool:** Vite 5 with Hot Module Replacement (HMR)
- **UI Components:** Material-UI (MUI) version 5.15
- **Styling:** Tailwind CSS 3.4
- **State Management:** React Context API
- **HTTP Client:** Axios
- **Charts:** Chart.js, Recharts
- **Icons:** Lucide React
- **Location:** `frontend/` directory

### Backend
- **Framework:** Django 3.2 with Django REST Framework
- **Language:** Python 3.11+
- **WebSockets:** Django Channels with Daphne
- **File Processing:** eppy (EnergyPlus IDF parsing), lxml
- **Location:** `backend/` directory

### Infrastructure
- **Database:** PostgreSQL 15 (port 5432)
- **Caching:** Redis 7 Alpine (port 6379)
- **Containerization:** Docker Compose for development and production
- **Simulation Engine:** EnergyPlus via NREL Docker image
- **Reverse Proxy (Production):** Nginx

## Repository Structure

```
epsm/
├── frontend/              # React TypeScript frontend
│   ├── src/
│   │   ├── components/   # React components (auth, baseline, simulation, results)
│   │   ├── context/      # React Context providers
│   │   ├── lib/          # API clients and utilities
│   │   ├── types/        # TypeScript type definitions
│   │   └── utils/        # Helper functions
│   ├── Dockerfile        # Development container
│   └── Dockerfile.prod   # Production container
├── backend/              # Django backend
│   ├── config/           # Django settings (settings.py, urls.py, asgi.py)
│   ├── simulation/       # Core simulation logic
│   ├── database/         # Database models for materials
│   ├── media/            # File uploads (IDF/EPW files, simulation results)
│   └── requirements.txt  # Python dependencies
├── scripts/              # Shell scripts (start.sh, stop.sh, status.sh, deploy.sh)
├── docs/                 # Documentation (markdown files)
├── tests/                # Test files
├── configs/              # Configuration files (symlinked to root for tool compatibility)
├── database/             # Database migrations and exports
├── .docker/              # Docker configurations
├── docker-compose.yml    # Development services
└── docker-compose.prod.yml  # Production services
```

### Important Note on Configuration Files
Configuration files like `tsconfig.json`, `vite.config.ts`, `tailwind.config.js`, `postcss.config.js`, and `eslint.config.js` are stored in the `configs/` directory but **symlinked to the root** for build tool compatibility. Don't modify the root symlinks—edit the actual files in `configs/`.

## Development Workflow

### Starting Development
1. Use `./scripts/start.sh` to start all Docker services
2. Frontend runs on http://localhost:5173 (Vite dev server with HMR)
3. Backend API runs on http://localhost:8000
4. Django admin available at http://localhost:8000/admin/ (admin/admin123)

### Making Code Changes

#### Frontend Changes
- Files in `frontend/src/` auto-reload via HMR
- TypeScript compilation happens automatically
- Follow existing component patterns (functional components with TypeScript)
- Use Material-UI components for consistency
- State management through React Context (see `context/` directory)

#### Backend Changes
- Django auto-reloads on Python file changes
- For model changes, run:
  ```bash
  docker-compose exec backend python manage.py makemigrations
  docker-compose exec backend python manage.py migrate
  ```
- Follow Django REST Framework conventions
- Use serializers for API responses

### Testing
- Run all tests: `./scripts/test.sh`
- Backend tests: `docker-compose exec backend python manage.py test`
- Frontend tests: `docker-compose exec frontend npm test`

## Code Conventions & Patterns

### Frontend
- **Components:** Functional components with TypeScript interfaces
- **File naming:** PascalCase for components (e.g., `BaselinePage.tsx`)
- **API calls:** Use `authenticatedFetch` from `lib/auth-api.ts`
- **Styling:** Combine Material-UI `sx` prop with Tailwind utility classes
- **Type safety:** Always define TypeScript interfaces for props and API responses

### Backend
- **API endpoints:** RESTful design with Django REST Framework
- **Models:** Use Django ORM, define in `models.py`
- **Views:** Class-based views or function-based with `@api_view` decorator
- **Serializers:** DRF serializers for data validation and transformation
- **File handling:** Store uploads in `media/` directory
- **WebSockets:** Django Channels consumers for real-time updates

### Database
- **Primary database:** `epsm_db` (application data, simulation tracking)
- **Materials database:** Separate schema for materials, constructions, schedules
- **Migrations:** Always create migrations for model changes
- **Credentials:** Default dev credentials are `epsm_user` / `epsm_secure_password`

## Common Tasks

### Adding a New API Endpoint
1. Define model in `backend/simulation/models.py` (if needed)
2. Create serializer in `backend/simulation/serializers.py`
3. Add view in `backend/simulation/views.py`
4. Register URL in `backend/config/urls.py`
5. Update frontend API client in `frontend/src/lib/`

### Adding a New Frontend Page
1. Create component in `frontend/src/components/`
2. Define TypeScript interfaces in `frontend/src/types/`
3. Add routing (if needed)
4. Use existing context providers for state management
5. Follow Material-UI theming and Tailwind utilities

### Running Simulations
- IDF files are parsed using `eppy` library
- Simulations run in Docker containers via EnergyPlus
- Results are stored in `backend/media/simulation_results/`
- WebSocket updates provide real-time progress

## Important Context

### File Processing
- **IDF files:** Parsed with `eppy` to extract geometry, zones, schedules
- **EPW files:** Weather data for simulations
- Upload size limits and validation are enforced

### Authentication
- Session-based authentication with Django
- CSRF protection enabled
- JWT tokens for API authentication
- Demo account available: demo@chalmers.se / demo123

### Docker Services
- All services run in Docker containers
- Use `docker-compose` commands for service management
- Volumes persist database and media files

### Environment Variables
- Defined in `.env` file (copy from `.env.example`)
- Default values work for development
- Production requires updating secrets and credentials

## Documentation

- **Getting Started:** `docs/GETTING_STARTED.md`
- **Development Guide:** `docs/DEVELOPMENT.md`
- **Architecture:** `docs/ARCHITECTURE.md`
- **Deployment:** `docs/DEPLOYMENT.md`
- **Contributing:** `CONTRIBUTING.md`

## Key Libraries & Dependencies

### Frontend Dependencies
- `@mui/material` - UI components
- `axios` - HTTP client
- `recharts`, `chart.js` - Data visualization
- `lucide-react` - Icons
- `react-router-dom` - Routing

### Backend Dependencies
- `djangorestframework` - REST API
- `channels` - WebSocket support
- `eppy` - EnergyPlus IDF parsing
- `psycopg2-binary` - PostgreSQL adapter
- `redis` - Caching

## Security & Best Practices

- Never commit secrets or credentials to the repository
- Use environment variables for configuration
- Input validation on both frontend and backend
- CSRF protection enabled
- SQL injection prevention via Django ORM
- File upload validation and size limits

## Troubleshooting

- **Port conflicts:** Frontend (5173), Backend (8000), DB (5432), Redis (6379)
- **Database issues:** Check with `docker-compose logs database`
- **Backend issues:** Check with `docker-compose logs backend`
- **Frontend build issues:** Clear `node_modules` and reinstall

## Project Goals

- Streamline EnergyPlus simulation workflow
- Enable batch processing of building scenarios
- Provide intuitive UI for non-experts
- Support large-scale building stock analysis
- Track environmental impact (GWP) and costs
- Enable collaboration and version control

## Contact & Support

- **Lead Developer:** Sanjay Somanath (sanjay.somanath@chalmers.se)
- **Principal Investigator:** Alexander Hollberg
- **Institution:** Chalmers University of Technology
- **Funding:** Swedish Energy Agency (Project ID P2024-04053)

---
> Source: [SB-Chalmers/epsm](https://github.com/SB-Chalmers/epsm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-13 -->
