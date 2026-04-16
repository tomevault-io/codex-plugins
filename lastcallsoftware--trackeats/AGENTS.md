# Copilot Workspace Instructions for TrackEats

## Overview
TrackEats is a full-stack nutrition tracking application designed as a portfolio project to demonstrate proficiency in modern web technologies. The project includes:
- **Frontend**: React (Vite, TypeScript, Chakra UI, Nginx)
- **Backend**: Python (Flask, SQLAlchemy, MySQL, JWT, Waitress)
- **Backend (Java)**: Spring Boot (Java 21, Maven, MySQL, JWT)
- **DevOps**: Docker, Docker Compose, GitHub Actions, AWS

## Key Conventions & Principles
- **Link, Don’t Embed**: Reference docs in `docs/` and per-app `README.md` files for details. Do not duplicate content.
- **Environment Management**: Use `.env` files for local dev, GitHub Secrets for CI/CD. Never commit secrets.
- **Branching**: Default to direct commits to `main` for solo/small team work. Use feature branches for larger/parallel efforts.
- **Testing**: Each backend has shell scripts in `bin/` for API and DB tests. See per-backend `README.md` for usage.
- **Build/Run**:
  - **Frontend**: `npm install` → `npm run dev` (local), `npm run build` (prod), see `frontend/README.md`.
  - **Backend-Python**: `python3 -m venv .venv` → `source .venv/bin/activate` → `pip install -r requirements.txt` → `python3 app.py` (local), see `backend-python/README.md`.
  - **Backend-Java**: `./mvnw spring-boot:run` (local), see `backend-java/HELP.md`.
- **Docker**: Each major component has a `Dockerfile`. Use Docker Compose for full stack. See root and per-app `README.md`.
- **Database**: MySQL, with setup/maintenance scripts in `bin/` and data in `data/`.
- **Secrets**: Never commit real secrets. Use `.env.example` as a template.

## Project Structure
- `frontend/` — React app (see `frontend/README.md`)
- `backend-python/` — Flask API (see `backend-python/README.md`)
- `backend-java/` — Spring Boot API (see `backend-java/HELP.md`)
- `docs/` — Design, setup, and technical notes
- `bin/` — Utility/test scripts (per backend)

## Common Pitfalls
- **Frontend env vars**: Must be prefixed with `VITE_` to be available in the app.
- **Backend env**: Activate Python venv before running backend scripts.
- **Docker**: Ensure Docker Desktop/Engine is running and you are logged in to Docker Hub for image operations.
- **MySQL**: Remote access requires correct server config and AWS security group rules (see `docs/technotes.md`).

## Documentation
- See `docs/design doc.md` for app architecture and design rationale.
- See `docs/setting-up.md` for environment setup (WSL, VS Code, Docker, Node, etc).
- See `docs/technotes.md` for SSH, MySQL, and WSL tips.
- See per-app `README.md` for build/run/test details.

## Example Prompts
- "How do I run the backend locally?"
- "How do I build and run the frontend in Docker?"
- "Where do I put environment variables for the frontend?"
- "How do I connect to the MySQL database?"

---
For advanced agent customization, consider creating applyTo-based instructions for each major directory (frontend, backend-python, backend-java) to further specialize agent behavior.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lastcallsoftware) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
