# Sys-Certificados AI Agent Instructions

## Project Architecture

This is a certificate management system with two main components:

1. **Backend (certi-backend)**:
   - Laravel-based REST API
   - Handles certificate generation, validation, and management
   - Key services in `app/Services/`:
     - `CertificateService.php`: Core certificate operations
     - `CertificateTemplateService.php`: Template management
     - `ValidationService.php`: Certificate validation logic
     - `ActivityService.php`: Activity tracking

2. **Frontend (certi-portal)**:
   - Angular-based web application
   - Provides user interface for certificate management
   - Located in `certi-portal/`

## Development Workflow

### Backend Development
```bash
# Install dependencies
cd certi-backend
composer install
php artisan key:generate

# Database setup
php artisan migrate
php artisan db:seed

# Start development server
php artisan serve
```

### Frontend Development
```bash
# Install dependencies
cd certi-portal
npm install

# Start development server
ng serve

# Run tests
ng test
```

## Project Conventions

### Backend Patterns
- Services layer (`app/Services/`) contains business logic
- Models (`app/Models/`) define database relationships and validation rules
- API responses use `ApiResponseTrait` for consistent formatting

### Frontend Patterns
- Angular CLI for component generation
- Component-based architecture
- End-to-end tests with Karma

## Key Integration Points

1. **API Endpoints**: Defined in `certi-backend/routes/api.php`
2. **Authentication**: Laravel Sanctum for API authentication
3. **File Storage**: Certificate documents stored using Laravel's filesystem
4. **Database**: Migrations in `certi-backend/database/migrations/`

## Common Development Tasks

1. **Adding New Certificate Features**:
   - Add model updates in `app/Models/Certificate.php`
   - Implement business logic in `CertificateService.php`
   - Create API endpoints in `routes/api.php`

2. **Template Management**:
   - Templates stored in `CertificateTemplate` model
   - Use `CertificateTemplateService` for operations

3. **Validation System**:
   - Implement validations in `ValidationService`
   - Each certificate can have multiple validations

## Important Files to Review
- `certi-backend/config/`: Configuration files
- `certi-backend/app/Services/`: Core business logic
- `certi-portal/src/app/`: Angular components
- `database/migrations/`: Database structure

---
> Source: [augenwx/sys_certificado](https://github.com/augenwx/sys_certificado) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-13 -->
