# Hub de Leitura - AI Development Guide

## Project Overview
Educational library management system built specifically for **QA learning and testing**. Every feature represents real-world QA scenarios with intentional complexity for practice.

## Architecture Patterns

### MVC Structure
- **Controllers**: `src/controllers/` - Business logic with Joi validation
- **Routes**: `src/routes/` - Express routes with Swagger documentation  
- **Middleware**: `src/middleware/auth.js` - JWT authentication + admin authorization
- **Database**: SQLite with connection in `config/db.js`

### Authentication Flow
- JWT tokens (1-hour expiration) with role-based access (`isAdmin` flag)
- Middleware chain: `authenticateToken` → `isAdmin` for admin routes
- Secret key defaults to `"admin@admin"` (configurable via `JWT_SECRET`)

## Key Conventions

### API Documentation
- **All routes must have Swagger docs** with QA-focused examples
- Use tags: `🔐 Autenticação`, `📚 Catálogo de Livros`, `📝 Reservas`, `👤 Usuários`, `🛠️ Admin`
- Include test scenarios in descriptions: "✅ Success case, ❌ Error case"

### Error Handling Pattern
```javascript
// Standard error responses with hints for QA testing
return res.status(401).json({ 
  message: "Token expirado",
  error: "EXPIRED_TOKEN", 
  hint: "Faça login novamente para obter um novo token"
});
```

### Validation with Joi
- Define schemas at controller top: `const bookSchema = Joi.object({...})`
- Always validate input before database operations
- Return descriptive validation errors for QA scenarios

## Database Schema
- **Users**: Authentication + role management (`isAdmin`)
- **Books**: Full bibliographic data + inventory tracking (`available_copies`)
- **Reservations**: User-book relationships with status tracking
- **Contacts**: Contact form submissions

## Development Workflow

### Database Management
```bash
# Reset database (common during development)
rm database/biblioteca.db
npm run db  # Recreates with sample data
npm start
```

### Test Credentials
- **Admin**: `admin@biblioteca.com` / `admin123`
- **User**: `usuario@teste.com` / `user123`

## Frontend Patterns

### Global Utilities (`public/js/utils.js`)
- Bootstrap-based alert system: `showSuccess()`, `showError()`, `showWarning()`
- Replaces browser alerts with styled notifications
- Auto-dismisses after 3 seconds (configurable)

### Authentication Headers
Frontend must include: `Authorization: Bearer {token}` for protected endpoints

## Critical Development Notes

### SQLite Connection
- Database must exist before server starts (check `config/db.js`)
- Connection failures stop server with detailed error messages
- Tables auto-created by `scripts/init_db.js`

### Swagger Integration
- Available at `/api-docs` with persistent authorization
- Custom interceptor adds tokens automatically from localStorage
- UI configured for QA workflow (expanded docs, filters enabled)

### Environment Setup
- Node.js 18+ required
- Port 3000 default (configurable via `PORT` env var)
- No external databases - SQLite for simplicity

## Common Issues & Solutions

### Port Conflicts
Use `lsof -ti:3000` and `kill -9 PID` to free port 3000

### Token Expiration
Tokens expire hourly - implement auto-renewal or clear error handling

### Database Corruption
Delete `database/biblioteca.db` and run `npm run db` to recreate

## File Structure Significance
- `data/books.json` - Sample data loaded by init script
- `public/` - Static assets served by Express
- `scripts/init_db.js` - Database schema + sample data creation
- `config/swaggerDef.js` - API documentation configuration

When working with this codebase, prioritize QA-friendly error messages, comprehensive API documentation, and maintainable test scenarios.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fabioaraujoqa) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
