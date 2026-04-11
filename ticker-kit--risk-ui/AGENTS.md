# Copilot Instructions for Risk UI

## Project Overview
This is a modern React frontend for managing investment portfolios with real-time price updates. The application is built with React 19 + Vite and integrates with a FastAPI backend through an event-driven microservices architecture using Redis pub/sub.

## Architecture & Technology Stack

### Frontend Stack
- **React 19** - UI framework with modern hooks and concurrent features
- **Vite** - Build tool and development server
- **Tailwind CSS v4.1** - Utility-first CSS framework
- **React Router v7** - Client-side routing with data loading
- **React Context** - State management for auth and global state

### Integration
- **FastAPI Backend** - RESTful API communication
- **Redis Pub/Sub** - Event-driven price updates
- **PostgreSQL** - Data persistence (via backend)
- **JWT Authentication** - Token-based auth with protected routes

## Key Features & Domain
- Portfolio position management (CRUD operations)
- Real-time ticker price updates
- Risk metrics calculation (volatility, Sharpe ratio, max drawdown)
- Ticker search and validation
- Event-driven price refresh system

## Code Patterns & Standards

### Component Structure
- Use functional components with hooks
- Prefer composition over inheritance
- Keep components focused and single-purpose
- Use custom hooks for shared logic

### State Management
- React Context for global state (auth, user data)
- Local state with useState for component-specific data
- Custom hooks for API calls and data fetching

### Styling Approach
- Use Tailwind CSS classes for styling
- Follow mobile-first responsive design
- Maintain consistent spacing and color schemes
- Use semantic class names when needed

### API Integration
- Centralized API calls in `/src/api/` directory
- Error handling with try/catch and user feedback
- Loading states for better UX
- Token-based authentication for protected endpoints

## File Organization
```
src/
├── api/           # API integration logic
├── components/    # Reusable UI components
├── contexts/      # React context providers
├── hooks/         # Custom React hooks  
├── pages/         # Route-level components
├── constants.js   # App-wide constants
└── main.jsx       # Application entry point
```

## Development Guidelines

### Code Quality
- Follow ESLint configuration (see `eslint.config.js`)
- Use meaningful variable and function names
- Add JSDoc comments for complex functions
- Keep functions small and focused

### Error Handling
- Implement proper error boundaries where needed
- Provide user-friendly error messages
- Handle network failures gracefully
- Use React Toast for user notifications

### Performance
- Use React.memo() for expensive components
- Implement proper loading states
- Avoid unnecessary re-renders
- Optimize bundle size (current build warns about chunk size)

## Bug Fixes & Design Changes

**Important:** As specified in the project requirements, do not exaggerate when making bug fixes or design changes. Keep modifications minimal and focused:

- Fix only the specific issue reported
- Maintain existing design patterns and UI consistency
- Avoid over-engineering solutions
- Test changes thoroughly before committing
- Preserve backward compatibility where possible

## Development Workflow

### Available Scripts
- `npm run dev` - Start development server (port 3000)
- `npm run build` - Production build
- `npm run lint` - Run ESLint
- `npm run preview` - Preview production build

### Environment Setup
- Copy `.env.example` to `.env.local` for local development
- Use `.env.docker` for Docker development
- Configure `VITE_API_URL` to point to your backend

### Docker Support
- Use `docker-compose up --build` for full stack development
- Individual Dockerfile available for containerized deployment
- Startup scripts available for different environments

## Testing Approach
- Focus on integration testing for user workflows
- Test API integration points thoroughly  
- Verify real-time update functionality
- Test authentication flows

## Common Patterns to Follow

### API Calls
```javascript
// Use try/catch with proper error handling
try {
  const response = await apiCall();
  // Handle success
} catch (error) {
  console.error('API Error:', error);
  // Show user-friendly error message
}
```

### Component Structure
```javascript
// Functional component with hooks
function ComponentName({ prop1, prop2 }) {
  const [localState, setLocalState] = useState(initialValue);
  
  // Custom hooks for complex logic
  const { data, loading, error } = useCustomHook();
  
  return (
    <div className="tailwind-classes">
      {/* JSX content */}
    </div>
  );
}
```

### Context Usage
```javascript
// Access context data
const { user, isAuthenticated } = useContext(AuthContext);
```

## Security Considerations
- Never expose API keys in frontend code
- Use environment variables for configuration
- Implement proper JWT token handling
- Validate user input before API calls
- Use HTTPS in production

## Performance Notes
- The current build produces large chunks (>500KB)
- Consider code splitting for route-based chunks
- Optimize imports to reduce bundle size
- Use dynamic imports for heavy components when appropriate

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ticker-kit)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/ticker-kit)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
