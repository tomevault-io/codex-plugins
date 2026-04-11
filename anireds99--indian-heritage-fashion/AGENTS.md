# Copilot Instructions for FashionBrand Project

## Project Overview
- **Project Name:** FashionBrand (Fashion E-commerce Platform)
- **Tech Stack:** Python Flask, SQLite, HTML/CSS/JavaScript
- **Purpose:** Indian heritage-inspired fashion e-commerce application with admin, user, and cart management

## Memory & Continuation Skill

### How to Use This File
When starting a new session, always:
1. Check the "## Current Status" section below to understand what was last worked on
2. Review the "## Known Issues & Fixes" section for any documented problems
3. Reference the "## Architecture" section to understand the codebase structure
4. Follow the guidelines in "## Development Practices" for consistency

### How to Update Progress
After completing work:
1. Update the "## Current Status" section with what was accomplished
2. Document any new issues or fixes in "## Known Issues & Fixes"
3. Note any changes to architecture in "## Architecture"
4. Add relevant commands to "## Useful Commands"

---

## Current Status

### Last Session Summary
**Date:** March 31, 2026  
**OS:** macOS  

**Last Completed Tasks:**
- [ ] Initial project setup
- [ ] Database schema creation
- [ ] Authentication system implementation
- [ ] Cart management system
- [ ] Admin dashboard
- [ ] Product gallery with Indian heritage designs
- [ ] Mockup generation for products

**Currently In Progress:**
- [ ] Document what you're working on here
- [ ] Next task to complete

**Blockers/Issues:**
- [ ] None currently

---

## Architecture

### Project Structure
```
FashionBrand/
├── app.py                 # Main Flask application
├── config/                # Configuration files
├── controllers/           # Business logic controllers
│   ├── admin_controller.py
│   ├── auth_controller.py
│   ├── cart_controller.py
│   └── user_controller.py
├── models/                # Database models
├── repositories/          # Data access layer
├── services/              # Business logic services
├── middleware/            # Custom middleware
├── templates/             # HTML templates
│   ├── auth/              # Login/Register
│   ├── cart/              # Cart & Checkout
│   ├── user/              # User dashboard
│   └── admin/             # Admin panel
├── static/                # CSS, JS, images
└── instance/fashion_brand.db  # SQLite database
```

### Key Components
- **Controllers:** Handle HTTP requests and route to services
- **Models:** SQLAlchemy ORM models for database entities
- **Repositories:** Data access abstraction layer
- **Services:** Business logic layer
- **Templates:** Jinja2 HTML templates with Tailwind/custom CSS

---

## Known Issues & Fixes

### Authentication Issues
- **Issue:** Login/Registration problems
- **Fix:** Check `debug_login.py` and `diagnose_login.py` for debugging
- **Status:** [ ] Resolved

### Image Generation
- **Issue:** Mockup generation may fail
- **Scripts:** `generate_images.py`, `generate_mockups.py`
- **Status:** [ ] Resolved

### Database
- **Location:** `/instance/fashion_brand.db`
- **Type:** SQLite3
- **Migrations:** Manual schema updates needed
- **Status:** [ ] Current

---

## Development Practices

### Code Style
- Follow PEP 8 for Python code
- Use descriptive variable and function names
- Add docstrings to functions and classes
- Keep functions focused and DRY (Don't Repeat Yourself)

### When Adding Features
1. Create database models in `models/`
2. Implement repository methods in `repositories/`
3. Add service logic in `services/`
4. Create controller endpoints in `controllers/`
5. Add templates in `templates/`
6. Update requirements.txt if adding new dependencies

### Testing & Debugging
- Use `flask_output.log` for debugging
- Run debug scripts before fixing issues
- Test authentication flows thoroughly
- Validate database queries in repository layer

### Database Workflow
1. Define model in `models/__init__.py`
2. Create repository methods for CRUD operations
3. Use repositories in services
4. Call services from controllers
5. Always validate input before database operations

---

## Useful Commands

### Running the Application
```bash
python app.py
# or
flask run
```

### Database Operations
```bash
# Open SQLite shell
sqlite3 instance/fashion_brand.db

# Backup database
cp instance/fashion_brand.db instance/fashion_brand.db.backup
```

### Debugging
```bash
python debug_login.py
python diagnose_login.py
python generate_images.py
python generate_mockups.py
```

### Requirements Management
```bash
pip freeze > requirements.txt
pip install -r requirements.txt
```

### Git Workflow
```bash
git add .
git commit -m "Feature: Brief description of changes"
git push
```

---

## Session Checkpoints

### Session 1 - March 31, 2026
- **Started:** Initial project discovery
- **Completed:** [ ] Document what was done
- **Next Steps:** 
  1. [ ] First next task
  2. [ ] Second next task
  3. [ ] Third next task

### Session 2 - [Date]
- **Started:** 
- **Completed:** 
- **Next Steps:** 

---

## Important Notes

### Before Starting Work
- Always read the most recent session checkpoint
- Check if there are any active issues to fix
- Review the current database state
- Verify all dependencies in requirements.txt

### Common Patterns
- **User Authentication:** Check `auth_controller.py` for examples
- **Cart Operations:** Reference `cart_controller.py`
- **Admin Operations:** Reference `admin_controller.py`
- **Database Access:** Follow patterns in `repositories/`

### Environment Variables
- Database: `instance/fashion_brand.db`
- Flask debug mode: Modify in `app.py`
- Secret key: Ensure it's set for production

---

## Continuation Strategy

When returning to this project:
1. **First:** Read "## Current Status" → "## Last Completed Tasks"
2. **Second:** Check "## Known Issues & Fixes" for any pending bugs
3. **Third:** Review the most recent "## Session Checkpoints"
4. **Fourth:** Start with the first unchecked task in "## Currently In Progress"
5. **Finally:** After completing work, update this file with new progress

---

**Last Updated:** March 31, 2026  
**Next Review:** When resuming work

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anireds99)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/anireds99)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
