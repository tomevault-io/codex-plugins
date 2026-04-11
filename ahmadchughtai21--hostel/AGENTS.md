# HOSTELZA Project - Development Guidelines

## Project Overview
Full-stack Django hostel listing and discovery platform with the following features:
- Django backend with PostgreSQL/MySQL support
- Tailwind CSS responsive frontend
- User roles: students, hostel owners, admins
- Google Maps API integration
- Image upload and management
- Advanced search and filtering

## Key Technologies
- **Backend**: Django 5.2.6, Python 3.12
- **Frontend**: Django Templates, Tailwind CSS
- **Database**: SQLite (dev), PostgreSQL (production)
- **Authentication**: Django built-in auth
- **Forms**: Django Crispy Forms

## Development Setup Completed
✅ Django project scaffolded with hostels app
✅ Database models created and migrated
✅ Admin interface configured
✅ Views, forms, and templates implemented
✅ Tailwind CSS integrated
✅ Development server running on http://localhost:8000

## Project Structure
```
hostel/
├── hostel_platform/     # Django project settings
├── hostels/            # Main Django app
├── templates/          # HTML templates
├── static/            # Static files
├── media/             # User uploads
└── manage.py          # Django management
```

## Available Commands
- Start server: `python manage.py runserver`
- Create migrations: `python manage.py makemigrations`
- Apply migrations: `python manage.py migrate`
- Create superuser: `python manage.py createsuperuser`
- Access admin: http://localhost:8000/admin (admin/admin123)

## Next Development Steps
1. Add more hostel listing templates
2. Implement Google Maps integration
3. Add image upload functionality
4. Create email notifications
5. Implement booking system (future)
6. Deploy to production server

Work through development systematically.
Keep communication concise and focused.
Follow Django best practices.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ahmadchughtai21)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/ahmadchughtai21)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
