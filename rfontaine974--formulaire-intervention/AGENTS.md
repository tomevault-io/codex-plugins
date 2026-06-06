# AI Coding Agent Instructions

## Project Overview
**formulaire-intervention** is a Flask web application for generating standardized French technical intervention reports. The application provides a single-page form interface where users select intervention details, and the app formats them into a standardized text report for copy-paste workflow.

## Architecture

### Single-File Design
- **app.py**: Contains HTML/CSS/JavaScript form (not traditional Python Flask code)
  - Served as static content via Flask
  - Embedded Tailwind CSS via CDN for styling
  - Inter font family via Google Fonts
  - Vanilla JavaScript handles all form logic (no framework)

### Data Model
The form captures 7 fields:
```
Type d'opération → select dropdown
Date → HTML5 date input (converted to "nuit du D au D+1" format)
Site(s) concerné(s) → text input (auto-uppercase)
Intervenant Principal → select dropdown (required)
Deuxième intervenant → select dropdown (optional, defaults to "-")
Détail de l'opération → textarea
Impact → optional text input (defaults to "Aucun impact")
```

### Key Business Logic

**Intervention Types** (valid_operations):
- Hard-coded list: "Autre HNO", "Insertion NRA", "Installation d'un DSLAM mobile", etc.
- Modify at line ~170 in `app.py`

**Technician Pairing Rules** (critical for backup logic):
**Technician List** (valid_intervenants):
- Hard-coded list at line ~12: includes roles (PQIS, CQIS, PDEM, CDEM)
- Update this list when adding new technicians
- Used for both "Intervenant Principal" and "Deuxième intervenant"
**Date Format**:
- Input: HTML5 `YYYY-MM-DD`
- Output: `Dans la nuit du DD/MM/YYYY au DD/MM/YYYY` (adds 1 day for night shift convention)
- Implementation: JavaScript `formatDate()` function uses local timezone

### Frontend Patterns

**Form Submission Flow**:
1. `submit` event → `preventDefault()`
2. Extract FormData → convert to object
3. Validate and transform (uppercase sites, date arithmetic)
4. Generate formatted text
5. Display in `<pre>` element with syntax highlighting
6. Show copy-to-clipboard button

**Copy to Clipboard**:
- Uses modern `navigator.clipboard` API
- Falls back to error message on failure
- Shows temporary "Copié !" confirmation

**Styling Convention**:
- Primary color: `#E80029` (red-500) via custom Tailwind config
- Hover variant: `#c40022` (darker red)
- Responsive: 2 columns on desktop (md:grid-cols-2), 1 on mobile
- Accessibility: Required fields marked with `<span class="text-red-500">*</span>`

## Development Workflows

### Local Development
```bash
# Install dependencies
pip install -r requirements.txt

# Run Flask app (development server)
python app.py
```

### Deployment (Heroku)
```bash
# Procfile.txt specifies entry point:
web: python app.py

# Start script exists (start.sh) for manual/CI use
```

### Modification Patterns

**Adding a new intervention type**:
```javascript
// Modify valid_operations array (line ~170)
const valid_operations = [
    "New Type Here",  // ← Add here
    "Autre HNO",
    ...
];
```

**Adding a new technician**:
```javascript
// Modify valid_intervenants array (line ~177)
const valid_intervenants = [
    "LASTNAME Firstname (TEAM_CODE)",  // ← Follow naming: NAME Surname (TEAM)
    ...
];
// Then update backup logic if new team introduced
```

**Updating form fields**:
- HTML markup: around line ~50-105
- Validation: HTML5 `required` attributes + JavaScript FormData
- Processing: Add to destructuring at line ~232

## External Dependencies & Integration

- **Flask 2.1.2**: Web server (likely serves the HTML form as static content)
- **Tailwind CSS**: CDN-loaded (no build step)
- **Werkzeug 2.1.2**: WSGI utility (Flask dependency)
- **No backend API**: Form generation is 100% client-side JavaScript

## Testing & Debugging

**No formal test suite exists**. Manual testing approach:

1. **Form validation**: Check HTML5 `required` attributes trigger on submit
2. **Backup logic**: Verify backup options filter correctly when changing intervenant
3. **Date formatting**: Confirm output shows "nuit du D au D+1"
4. **Clipboard**: Test across browsers (use console for errors)

**Common Issues**:
- Clipboard API fails on non-HTTPS or sandboxed contexts → error message displayed
- Date parsing with `new Date(date + 'T00:00:00')` assumes local timezone
- Missing fields in `valid_operations` or `valid_intervenants` = blank dropdowns

## Project Philosophy

This is a **minimal, pragmatic** form generator:
- No JavaScript framework complexity
- No backend state management
- No database (all hardcoded dropdowns)
- Client-side only: users copy text and paste elsewhere
- Static HTML served as Python file (quirky but works)

**Convention**: Keep it simple. Add features only as form fields, not API endpoints.

---
> Source: [rfontaine974/formulaire-intervention](https://github.com/rfontaine974/formulaire-intervention) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-06 -->
