# GitHub Copilot Instructions for PrimeTrade Project

## Project Overview
This is a full-stack REST API application with authentication and role-based access control. The project consists of a Node.js/Express backend and a Next.js frontend, with MongoDB for data persistence and Redis for caching.

## Architecture
- **Backend**: Node.js + Express.js (RESTful API)
- **Frontend**: Next.js 16.2.1 with React 19 + Tailwind CSS
- **Database**: MongoDB with Mongoose ODM
- **Cache**: Redis (ioredis)
- **Authentication**: JWT (access tokens + refresh tokens)
- **Deployment**: Docker + Docker Compose

## Code Style & Conventions

### Backend (Node.js)
- Use ES6+ syntax with async/await for asynchronous operations
- Use modular architecture: separate concerns into models, controllers, routes, middleware, utils
- Follow RESTful API naming conventions:
  - Collections: plural nouns (`/api/v1/tasks`, `/api/v1/notes`)
  - Resources: singular with ID (`/api/v1/tasks/:id`)
- Error handling: Always use try-catch blocks with proper error responses
- Use descriptive function names: `createTask`, `getUserById`, `validateUserInput`
- Constants in UPPER_SNAKE_CASE, variables in camelCase
- Add JSDoc comments for all exported functions
- Never commit sensitive data (use .env for secrets)

### Frontend (Next.js/React)
- Use functional components with React Hooks
- Follow Next.js 16+ conventions: App Router, Server Components where applicable
- Component naming: PascalCase for components (`TaskCard.js`, `AuthForm.js`)
- Use Tailwind CSS for styling (avoid inline styles)
- Create reusable components in `src/app/components/`
- Use client-side state management with React Context for auth
- API calls: centralize in `src/app/lib/api.js`
- Handle loading states and errors gracefully with user feedback

### Security Best Practices
- Never log sensitive data (passwords, tokens, API keys)
- Always validate and sanitize user inputs
- Use bcrypt for password hashing (salt rounds: 10)
- Implement rate limiting on auth endpoints
- Use CORS with specific origins (no wildcards in production)
- Add security headers with Helmet.js
- Store JWT tokens securely (httpOnly cookies preferred, or localStorage with XSS protection)
- Validate JWT tokens on every protected route
- Implement proper RBAC: check user roles before allowing actions

### Database (MongoDB)
- Use Mongoose schemas with validation
- Add indexes on frequently queried fields (userId, email, status)
- Use timestamps: `{ timestamps: true }`
- Implement soft deletes when appropriate
- Use transactions for operations affecting multiple collections
- Always handle connection errors and implement reconnection logic

### API Versioning
- Version all APIs: `/api/v1/...`
- Maintain backward compatibility when adding v2
- Document breaking changes clearly

### Error Handling
- Return consistent error format:
  ```json
  {
    "success": false,
    "error": "Error message",
    "statusCode": 400
  }
  ```
- Use appropriate HTTP status codes:
  - 200: Success
  - 201: Created
  - 400: Bad Request (validation errors)
  - 401: Unauthorized (missing/invalid token)
  - 403: Forbidden (insufficient permissions)
  - 404: Not Found
  - 500: Internal Server Error
- Log errors server-side with context (don't expose stack traces to clients in production)

### Testing & Validation
- Validate all inputs using express-validator
- Test authentication flow thoroughly
- Verify role-based access control
- Test CRUD operations for all entities
- Verify caching behavior
- Test Docker deployment before production

### Documentation
- Document all API endpoints with Swagger/JSDoc
- Keep README files up to date
- Include setup instructions, environment variables, and usage examples
- Add inline comments for complex logic only (code should be self-explanatory)

### Performance & Scalability
- Use Redis caching for frequently accessed data
- Implement pagination for list endpoints (default: 20 items per page)
- Use database indexes for optimization
- Keep stateless API design for horizontal scaling
- Monitor and log performance metrics

### Git Workflow
- Write clear commit messages: "feat: add user authentication" or "fix: resolve task deletion bug"
- Never commit `.env` files or `node_modules`
- Keep commits focused (one feature/fix per commit)
- Use branches for features: `feature/task-crud`, `fix/auth-bug`

## Common Patterns

### Controller Pattern
```javascript
exports.createResource = async (req, res, next) => {
  try {
    const resource = await Resource.create(req.body);
    res.status(201).json({
      success: true,
      data: resource
    });
  } catch (error) {
    next(error); // Pass to error handler middleware
  }
};
```

### Protected Route Pattern
```javascript
router.get('/tasks', auth, getTasks); // Requires authentication
router.delete('/users/:id', auth, restrictTo(['admin']), deleteUser); // Requires admin role
```

### Frontend API Call Pattern
```javascript
const fetchTasks = async () => {
  try {
    setLoading(true);
    const response = await api.get('/tasks');
    setTasks(response.data);
  } catch (error) {
    toast.error(error.message);
  } finally {
    setLoading(false);
  }
};
```

### Redis Caching Pattern
```javascript
const cacheKey = `tasks:user:${userId}`;
let cachedData = await redis.get(cacheKey);

if (cachedData) {
  return JSON.parse(cachedData);
}

const data = await Task.find({ userId });
await redis.setex(cacheKey, 120, JSON.stringify(data)); // 2 min TTL
return data;
```

## File Structure Reference

### Backend (`server/`)
```
src/
├── config/           # Configuration files (db, redis, jwt)
├── models/           # Mongoose models (User, Task, Note)
├── controllers/      # Business logic (authController, taskController, etc.)
├── routes/           # Express routes (authRoutes, taskRoutes, etc.)
├── middleware/       # Custom middleware (auth, authorize, validators, errorHandler)
├── utils/            # Helper functions
└── app.js            # Express app initialization
```

### Frontend (`client/`)
```
src/app/
├── (auth)/           # Auth pages (login, register)
├── dashboard/        # Protected dashboard pages
│   ├── tasks/        # Task management
│   ├── notes/        # Note management
│   └── admin/        # Admin panel
├── components/       # Reusable UI components
├── context/          # React contexts (AuthContext)
└── lib/              # Utilities (api, toast)
```

## Troubleshooting Tips
- MongoDB connection issues: Check connection string, network access, and MongoDB service status
- Redis connection issues: Verify Redis is running (`redis-cli ping`)
- JWT errors: Check token expiration, secret key consistency, and token format
- CORS errors: Verify frontend origin in backend CORS config
- Docker issues: Check port conflicts, environment variables, and service dependencies

## When in Doubt
- Follow REST principles and HTTP standards
- Prioritize security and validation
- Keep code DRY (Don't Repeat Yourself)
- Write self-documenting code with clear naming
- Test thoroughly before considering a feature complete

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/codedpool)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/codedpool)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
