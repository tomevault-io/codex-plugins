# Copilot Instructions: Tetris Game

## Architecture Overview

This is a full-stack Tetris game with Express.js/TypeScript backend and Angular 19 standalone components frontend:

- **Backend** (`/backend`): Node.js Express server with Prisma ORM, PostgreSQL database, JWT authentication
- **Frontend** (`/frontend`): Angular 19 with standalone components, full routing configured
- **Database**: PostgreSQL with Prisma migrations, custom Prisma client output to `src/generated/prisma`
- **Features**: Player leaderboard, score tracking, player switching, game statistics

## Key Development Workflows

```bash
# Backend development
cd backend && npm run dev  # ts-node-dev with auto-restart
cd backend && npx prisma migrate dev  # Apply database migrations
cd backend && npx prisma generate     # Regenerate Prisma client

# Frontend development  
cd frontend && ng serve  # Development server on http://localhost:4200
cd frontend && ng generate component component-name  # Generate new component
cd frontend && ng build  # Production build

# Database management
cd backend && npx prisma studio  # Database GUI
cd backend && npx prisma db push  # Push schema without migration
```

## Project-Specific Conventions

### Backend Patterns
- **Prisma client location**: Custom output to `src/generated/prisma` (not default node_modules)
- **Auth pattern**: JWT tokens named `tokenA` (not `token` or `access_token`)
- **Controller structure**: Static methods in classes (e.g., `AuthController.register`)
- **Error handling**: Throw `Error` with message, catch in controller, return `{ error: message }`
- **API structure**: Direct route handlers in `index.ts` + separate route files in `/routes`

### Frontend Patterns  
- **Component architecture**: All components are standalone (imports array, no NgModules)
- **Routing**: Full Angular Router with routes for auth, game, leaderboard, game-over
- **Styling**: SCSS files, component-scoped styles with responsive design
- **Game logic**: Tetris engine in `game-board.component.ts` with tetromino definitions in separate file
- **Internationalization**: Comments and some code in Ukrainian
- **Avatar system**: Emoji-based avatars from predefined array
- **State management**: Score/level tracking with automatic saving to database
- **Player management**: Logout functionality and player switching in header

### Database Schema
```prisma
model User {
  id       Int     @id @default(autoincrement())  
  username String  @unique
  password String  # bcrypt hashed
  avatar   String  # emoji string
  createdAt DateTime @default(now())
  scores   Score[] # One-to-many relation with scores
}

model Score {
  id        Int      @id @default(autoincrement())
  userId    Int
  user      User     @relation(fields: [userId], references: [id])
  score     Int      # Game score
  level     Int      # Achieved level
  playedAt  DateTime @default(now())
  
  @@index([userId])
  @@index([score])
}
```

## Critical Integration Points

### Authentication Flow
1. Frontend calls `/auth/register` or `/auth/login`  
2. Backend returns `{ tokenA: "jwt_token" }`
3. Frontend stores token and sends as `Authorization: Bearer {token}`
4. Protected routes use `authenticateToken` middleware

### API Endpoints
**Authentication:**
- `POST /auth/register` - Register with username, password, avatar
- `POST /auth/login` - Login returning tokenA
- `GET /protected` - Example protected endpoint  
- `GET /auth/me` - Get current user info (via auth routes)

**Scores & Leaderboard:**
- `POST /scores/save` - Save game result (protected)
- `GET /scores/leaderboard` - Get top players (public)
- `GET /scores/my-scores` - Get user's best scores (protected)
- `GET /scores/my-stats` - Get user statistics (protected)
- `GET /scores/my-ranking` - Get user's ranking position (protected)

### Cross-Origin Setup
- Backend CORS enabled for all origins
- Frontend development server: `http://localhost:4200`
- Backend API server: `http://localhost:3001`
- API base URL hardcoded in `AuthService`

### Game Components
- `GameBoardComponent`: Main Tetris engine with keyboard controls
- `Tetromino` types: Defined in separate file with shape coordinates and colors  
- Game state: Score, level, speed, grid managed in component
- Controls: Arrow keys, spacebar, R for restart
- `LeaderboardComponent`: Display top players with statistics and ranking
- `GameOverComponent`: End game screen with score saving and leaderboard access
- `HeaderComponent`: Navigation with leaderboard access and player switching
- `AuthPageComponent`: Login/registration with avatar selection

### Frontend Routes
- `/` - Authentication page (login/register)
- `/base` - Main game screen
- `/game-over` - Game over screen with score
- `/leaderboard` - Players leaderboard and statistics

### Player Management
- Automatic score saving after game completion
- Player logout/switching via header button
- Personal statistics and ranking display
- Top players leaderboard with rankings

## Important Files to Understand
**Backend:**
- [`backend/src/index.ts`](backend/src/index.ts) - Express app setup and main routes
- [`backend/prisma/schema.prisma`](backend/prisma/schema.prisma) - Database schema with User and Score models
- [`backend/src/services/score.service.ts`](backend/src/services/score.service.ts) - Score operations and leaderboard logic
- [`backend/src/controllers/score.controller.ts`](backend/src/controllers/score.controller.ts) - Score API endpoints
- [`backend/src/routes/scores.routes.ts`](backend/src/routes/scores.routes.ts) - Score routing configuration

**Frontend Core:**
- [`frontend/src/app/game-board/game-board.component.ts`](frontend/src/app/game-board/game-board.component.ts) - Core game logic
- [`frontend/src/app/game-board/tetris/tetromino.ts`](frontend/src/app/game-board/tetris/tetromino.ts) - Game piece definitions
- [`frontend/src/main.ts`](frontend/src/main.ts) - App bootstrap with routing configuration
- [`frontend/src/app/services/auth.service.ts`](frontend/src/app/services/auth.service.ts) - Authentication and logout logic

**Frontend Components:**
- [`frontend/src/app/leaderboard/leaderboard.component.ts`](frontend/src/app/leaderboard/leaderboard.component.ts) - Leaderboard display and statistics
- [`frontend/src/app/game-over/game-over.component.ts`](frontend/src/app/game-over/game-over.component.ts) - Game end with score saving
- [`frontend/src/app/header/header.component.ts`](frontend/src/app/header/header.component.ts) - Navigation with player switching
- [`frontend/src/app/services/score.service.ts`](frontend/src/app/services/score.service.ts) - Score API integration

**Assets:**
- [`frontend/src/assets/emoji-avatars.ts`](frontend/src/assets/emoji-avatars.ts) - Available user avatars

---
> Source: [IhorChornyj/Tetris-Fullstack-APPlication](https://github.com/IhorChornyj/Tetris-Fullstack-APPlication) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-27 -->
