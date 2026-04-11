# Copilot Instructions for NT208 - Chính quyền điện tử PKI

This document provides guidance for AI coding agents working on this project. It outlines the architecture, workflows, and conventions specific to this codebase to ensure productivity and alignment with project goals.

## Project Overview

This project implements an electronic government system leveraging Public Key Infrastructure (PKI). It consists of a **backend** built with Django and Django REST Framework (DRF) and a **frontend** built with React.

### Key Components

1. **Backend**:
   - Located in the `backend/` directory.
   - Major apps:
     - `authentication`: Handles user authentication and authorization.
     - `hoso`: Manages document-related operations.
     - `social_auth`: Integrates social login (Google, Facebook, etc.).
   - Configuration files:
     - `backend/chinhquyendtPKI_djangov2/settings.py`: Main Django settings.
   - Database: PostgreSQL is used as the database backend.

2. **Frontend**:
   - Located in the `frontend/` directory.
   - Key directories:
     - `src/components/`: Contains reusable React components.
     - `src/container/`: Defines page-level components and templates.
     - `src/redux/`: Manages global state using Redux.

3. **Static Files**:
   - Backend static files: `backend/static/`
   - Frontend assets: `frontend/public/` and `frontend/src/assets/`

### Data Flow
- The frontend communicates with the backend via REST APIs.
- Authentication uses token-based mechanisms (e.g., JWT).
- Social authentication is integrated via `social_auth`.

## Developer Workflows

### Backend

1. **Setup**:
   - Install dependencies:
     ```bash
     pip install -r backend/requirements.txt
     ```
   - Configure PostgreSQL:
     ```sql
     CREATE USER <username> WITH PASSWORD '<password>';
     CREATE DATABASE <dbname> OWNER <username>;
     ```

2. **Run the Server**:
   ```bash
   python manage.py makemigrations
   python manage.py migrate
   python manage.py runserver
   ```

3. **Create a Superuser**:
   ```bash
   python manage.py createsuperuser
   ```

4. **Testing**:
   - Tests are located in `tests.py` files within each app.
   - Run tests:
     ```bash
     python manage.py test
     ```

### Frontend

1. **Setup**:
   - Install dependencies:
     ```bash
     npm install
     ```

2. **Run the Development Server**:
   ```bash
     npm start
   ```

3. **Build for Production**:
   ```bash
     npm run build
   ```

## Project-Specific Conventions

1. **Backend**:
   - Follow Django's app structure.
   - Use serializers for data validation and transformation.
   - Place utility functions in `utils.py`.

2. **Frontend**:
   - Use functional components with hooks.
   - Organize styles in `scss/`.
   - Follow the container-component pattern.

3. **General**:
   - Use meaningful commit messages.
   - Document code with comments where logic is complex.

## Integration Points

1. **Social Authentication**:
   - Implemented in `backend/social_auth/`.
   - Supports Google and Facebook login.

2. **PDF Signing**:
   - Managed in `backend/signpdf.py`.

3. **Pagination**:
   - Custom pagination logic in `backend/hoso/paginator.py`.

## External Dependencies

- **Backend**:
  - `django-cors-headers`: Handles CORS.
  - `djoser`: Simplifies user authentication.
  - `django-x509`: Manages PKI certificates.

- **Frontend**:
  - React, Redux, and SCSS for styling.

## References

- [Django REST Framework](https://www.django-rest-framework.org/)
- [React Documentation](https://reactjs.org/docs/getting-started.html)
- [PostgreSQL Documentation](https://www.postgresql.org/docs/)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/phmngcthanh)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/phmngcthanh)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
