# GitHub Copilot Instructions for EcoBuddy

## Project Overview

EcoBuddy is a full-stack energy-saving companion app designed for teens to track their energy consumption, complete challenges, and interact with a virtual pet mascot. The application features user authentication, data persistence, social features, and gamification elements.

## Architecture

This is a monorepo containing:
- **Frontend**: React 18 + Vite 5 application with Tailwind CSS
- **Backend**: Express.js REST API with PostgreSQL database
- **Database**: PostgreSQL with JWT authentication

```
Frontend (Vite) → Backend API (Express) → PostgreSQL Database
```

## Tech Stack

### Frontend
- **Framework**: React 18 with functional components and hooks
- **Build Tool**: Vite 5
- **Routing**: React Router DOM v6
- **Styling**: Tailwind CSS 3 (mobile-first design)
- **Charts**: Chart.js 4 with React Chart.js 2
- **State Management**: React Context API (AuthContext)

### Backend
- **Framework**: Express.js with ES modules (`"type": "module"`)
- **Database**: PostgreSQL with `pg` driver
- **Authentication**: JWT (jsonwebtoken) + bcrypt for password hashing
- **Validation**: express-validator
- **Security**: CORS, express-rate-limit
- **Environment**: dotenv for configuration

## Code Style and Conventions

### General Guidelines
- Use ES6+ syntax with ES modules (import/export)
- Prefer functional components with hooks over class components
- Use arrow functions for component definitions
- Follow mobile-first responsive design principles
- Write descriptive variable and function names

### React Components
- Place components in `/src/components/` for reusable UI elements
- Place pages in `/src/pages/` for route components
- Use `.jsx` extension for components
- Export components as default exports
- Use React hooks (useState, useEffect, useContext) appropriately

### Styling
- Use Tailwind CSS utility classes for styling
- Brand colors are defined in `tailwind.config.cjs`:
  - Primary: `brand-primary` (#10b981 - green)
  - Accent: `brand-accent` (#6ee7b7 - light green)
- Design is mobile-first with bottom navigation bar
- Keep responsive design in mind for all components

### Backend API
- Use ES module syntax (import/export)
- Place routes in `/backend/src/routes/`
- Place models in `/backend/src/models/`
- Use middleware for authentication (`/backend/src/middleware/auth.js`)
- Validate all inputs using express-validator
- Return consistent JSON responses with appropriate status codes
- Use async/await for database operations

### Database
- Use PostgreSQL connection pool from `/backend/src/config/database.js`
- Run migrations via `npm run db:migrate` in backend directory
- Always use parameterized queries to prevent SQL injection
- Handle database errors gracefully

## Development Workflow

### Frontend Development
```bash
# Install dependencies
npm install

# Start dev server (runs on http://localhost:5173)
npm run dev

# Build for production
npm run build

# Preview production build
npm run preview
```

### Backend Development
```bash
cd backend

# Install dependencies
npm install

# Set up environment variables (see .env.example)
# Create PostgreSQL database: createdb energyteen

# Run migrations
npm run db:migrate

# Start dev server with watch mode (runs on http://localhost:3001)
npm run dev

# Start production server
npm start
```

### Environment Variables

**Frontend** (`.env` in root):
```env
VITE_API_URL=http://localhost:3001/api
```

**Backend** (`.env` in `/backend/`):
```env
DATABASE_URL=postgresql://username:password@localhost:5432/energyteen
JWT_SECRET=your-secret-key-here
NODE_ENV=development
PORT=3001
FRONTEND_URL=http://localhost:5173
```

## Key Features and Components

### Authentication System
- User signup/login with JWT tokens
- AuthContext provides authentication state across the app
- Protected routes require authentication
- Tokens stored in localStorage

### Pages
- **Home** (`/`): EcoBuddy mascot with customization, seeds, streaks, levels
- **Analytics** (`/analytics`): Energy usage tracking with charts
- **Leaderboard** (`/leaderboard`): Friends rankings and social features
- **Tasks** (`/tasks`): Challenges that users can complete for rewards
- **Profile** (`/profile`): User profile and settings
- **Login/Signup**: Authentication pages

### Currency System
- **Seeds**: Primary currency earned by completing tasks
- Users can feed/play with EcoBuddy using seeds
- Seeds shown on leaderboard rankings

### Data Visualization
- Use Chart.js for all charts (line charts, pie/doughnut charts)
- Components: `ChartLine.jsx`, `ChartPie.jsx`, `KpiCard.jsx`
- Show real data only, zeros for new users

## Testing and Quality Assurance

### Before Committing
1. Test the feature locally in dev mode
2. Test both frontend and backend if changes affect API
3. Verify mobile responsiveness (the app is mobile-first)
4. Check console for errors or warnings
5. Ensure no sensitive data (API keys, passwords) are committed

### Manual Testing
- Test authentication flow (signup, login, logout)
- Verify data persistence across page refreshes
- Check that API endpoints return expected data
- Test error handling for network failures

## Common Tasks

### Adding a New Page
1. Create component in `/src/pages/`
2. Add route in `/src/App.jsx`
3. Add navigation link in `/src/components/NavBottom.jsx` if needed
4. Ensure mobile-first responsive design

### Adding a New API Endpoint
1. Create or modify route in `/backend/src/routes/`
2. Add validation using express-validator
3. Use authentication middleware if protected
4. Update corresponding frontend service in `/src/services/api.js`
5. Test with both authenticated and unauthenticated requests

### Adding a Database Table
1. Create migration in `/backend/src/config/migrate.js`
2. Create model in `/backend/src/models/`
3. Run `npm run db:migrate` to apply changes
4. Update relevant API endpoints

### Styling Changes
1. Use Tailwind utility classes
2. Maintain mobile-first approach
3. Use brand colors: `brand-primary`, `brand-accent`
4. Test on different screen sizes

## Deployment

The application is deployed on Render with three services:
1. **Frontend**: Static site (Vite build)
2. **Backend**: Web service (Node.js)
3. **Database**: PostgreSQL managed database

See `DEPLOY_TO_RENDER.md` for complete deployment instructions.

## Important Files

- `package.json`: Frontend dependencies and scripts
- `vite.config.js`: Vite configuration
- `tailwind.config.cjs`: Tailwind CSS configuration
- `src/App.jsx`: Main app component with routing
- `src/context/AuthContext.jsx`: Authentication state management
- `backend/src/server.js`: Express server entry point
- `backend/src/config/database.js`: Database connection pool
- `backend/src/config/migrate.js`: Database migrations
- `render.yaml`: Render deployment blueprint

## Security Considerations

- Never commit `.env` files or sensitive credentials
- Always use parameterized queries for database operations
- Validate and sanitize all user inputs
- Use CORS properly to restrict API access
- Hash passwords with bcrypt before storing
- Use JWT tokens for authentication
- Implement rate limiting on API endpoints
- Keep dependencies updated for security patches

## Troubleshooting

### Frontend Issues
- Clear Vite cache: `rm -rf node_modules/.vite`
- Rebuild: `npm run build`
- Check that `VITE_API_URL` is set correctly

### Backend Issues
- Verify database connection: check `DATABASE_URL`
- Run migrations: `npm run db:migrate`
- Check logs for error messages
- Verify JWT_SECRET is set

### Common Errors
- **CORS errors**: Check `FRONTEND_URL` in backend `.env`
- **Database connection failed**: Verify PostgreSQL is running and credentials are correct
- **401 Unauthorized**: Check JWT token validity and authentication middleware
- **Module not found**: Run `npm install` in appropriate directory

## Additional Resources

- Project documentation in repository root:
  - `README.md`: Main project documentation
  - `DEPLOY_TO_RENDER.md`: Deployment guide
  - `BACKEND_DEPLOYMENT.md`: Backend-specific deployment
  - `QUICKSTART.md`: Quick start guide

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jlisiuottawa) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
