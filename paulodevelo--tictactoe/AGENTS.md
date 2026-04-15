# AI Agent Instructions for Tic-Tac-Toe Multiplayer Project

## Project Architecture Overview

This is a **full-stack real-time multiplayer Tic-Tac-Toe** with strict separation of concerns:
- **Backend**: Node.js/TypeScript with Socket.IO for real-time communication (`backend/`)
- **Frontend**: Angular 20 application with WCAG 2.1 AA accessibility (`unicorn-tictactoe/`)
- **Dual Game Modes**: Local play + real-time online multiplayer via WebSocket

## Critical Development Patterns

### Functional Programming (Backend)
The backend follows **strict functional patterns** - all models are immutable:
```typescript
// ✅ Correct: Return new objects, never mutate
const updatedRoom = addPlayerToRoom(room, player);
const newGameState = makeMove(gameState, position, symbol);

// ❌ Never mutate existing objects
room.players.push(player); // DON'T DO THIS
```

### State Management (Frontend)
Uses **RxJS BehaviorSubject pattern** for reactive state:
```typescript
// GameService pattern for all state
private gameStateSubject = new BehaviorSubject<GameState>(initialState);
public gameState$ = this.gameStateSubject.asObservable();
```

### WebSocket Event Architecture
**Room-based broadcasting** with specific event naming:
- Client→Server: `room:join`, `game:move`, `player:ready`
- Server→Client: `room:updated`, `game:started`, `player:disconnected`
- **Always** emit both specific events AND `room:updated` for state sync

## Essential Commands & Workflows

### Development Setup
```bash
# Backend (Terminal 1)
cd backend && npm install && npm run dev  # Port 3021

# Frontend (Terminal 2)  
cd unicorn-tictactoe && npm install && npm start  # Port 4200
```

### Testing Strategy
- **Backend**: Jest with 100% coverage requirement
- **Frontend**: Jasmine/Karma with 95%+ coverage target
- **Test Files**: `backend/src/__tests__/` and `unicorn-tictactoe/src/app/**/*.spec.ts`

### Production Deployment
Uses **PM2 + Nginx** with zero-downtime reloads:
```bash
cd backend && npm run deploy  # Automated deployment script
```

## Key File Relationships

### Data Flow Architecture
1. **Models** (`backend/src/models/`): Pure functions, immutable data structures
2. **Services** (`backend/src/services/RoomService.ts`): Business logic singleton
3. **WebSocket** (`backend/src/server.ts`): Event handling with room management
4. **Frontend Services** (`unicorn-tictactoe/src/app/services/`): RxJS reactive patterns

### Critical Dependencies
- **Real-time**: Backend `socket.io` ↔ Frontend `socket.io-client` 
- **State**: Frontend uses BehaviorSubject, Backend uses Map<string, Room>
- **Accessibility**: All components must maintain ARIA labels and keyboard navigation

## Project-Specific Conventions

### Symbol Assignment Logic
Players get symbols based on join order: **first player = 'X', second player = 'O'**
- Backend auto-starts games when room is full (2 players)
- Frontend handles both local turns and WebSocket synchronization

### Error Handling Pattern
```typescript
// Backend: Emit errors to specific socket
socket.emit('error', { message: 'Room not found' });

// Frontend: 5-second auto-clear error messages
this.connectionError = error;
setTimeout(() => this.connectionError = null, 5000);
```

### Environment Configuration
- **Backend**: `.env` with `PORT`, `CORS_ORIGIN`, `NODE_ENV`  
- **Frontend**: `src/environments/` for `apiUrl` configuration
- **CORS**: Must match frontend URL exactly for WebSocket connections

## Accessibility Requirements (WCAG 2.1 AA)

All new components must include:
- **ARIA labels**: `aria-label`, `role` attributes
- **Keyboard navigation**: Tab, Enter, Space key support
- **Live regions**: `aria-live="polite"` for game state announcements
- **Focus management**: Clear visual focus indicators

## Testing Patterns

### Backend Test Structure
```typescript
// RoomService.test.ts pattern
describe('RoomService', () => {
  describe('methodName', () => {
    it('should handle normal case', () => {});
    it('should throw error for invalid input', () => {});
  });
});
```

### Frontend Test Structure  
```typescript
// Component.spec.ts pattern
beforeEach(() => {
  TestBed.configureTestingModule({...});
  service = TestBed.inject(ServiceName);
});
```

## Common Pitfalls to Avoid

1. **Never mutate** backend model objects - always return new instances
2. **Always emit** `room:updated` after any room state change in WebSocket events
3. **CORS mismatches** between frontend URL and backend `CORS_ORIGIN` break WebSocket
4. **Missing ARIA labels** will fail accessibility audits
5. **Frontend environment** configuration must match backend URL for online mode

## File Modification Priority

When implementing features:
1. **Models first**: Update data structures in `backend/src/models/`
2. **Service logic**: Implement business logic in `RoomService.ts`
3. **WebSocket events**: Add event handlers in `server.ts`
4. **Frontend services**: Update Angular services for state management
5. **Components last**: Update UI components and templates

Use this knowledge to maintain the project's functional programming patterns, real-time synchronization, and accessibility standards.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/PauloDevelo) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
