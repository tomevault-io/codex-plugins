# BoatOwner Development Guide - GitHub Copilot Instructions

**⚠️ CRITICAL: Read this entire document before starting any development work.**

This guide provides comprehensive instructions for GitHub Copilot and developers working on the BoatOwner project. Following these instructions ensures consistency, quality, and adherence to project standards.

---

## 🚨 MANDATORY PROJECT DOCUMENTATION

### **MUST READ BEFORE ANY DEVELOPMENT:**
1. **`/DEVELOPMENT_RULES.md`** - Strict development rules and feature lifecycle
2. **`/README.md`** - Project overview, setup, and architecture
3. **`/ui/mobile/BoatOwner/FRONTEND_RULES.md`** - Frontend implementation guidelines
4. **`/doc/architecture/backend-architectural-patterns.mdc`** - Backend architecture patterns
5. **`/doc/architecture/mobile-ui-architectural-patters.md`** - Mobile UI architecture patterns

### **Project Resources:**
- **Trello Board**: https://trello.com/b/7GH6OZzf/boatowner
- **Domain Entities**: User, Boat, Logs, Tasks, Expenses
- **Tech Stack**: Node.js/Express, React Native/Expo, PostgreSQL, Prisma, TypeScript

---

## 🏗️ PROJECT ARCHITECTURE OVERVIEW

### **Backend Architecture (Express + Prisma)**
```
server/
├── controllers/     # HTTP request/response handling (NO business logic)
├── services/        # Business logic (controllers call services)
├── routers/         # Route definitions
├── middleware/      # Auth, validation, error handling
├── interfaces/      # TypeScript interfaces
├── prisma/         # Database schema and ORM
└── __tests__/      # Unit and integration tests
```

### **Frontend Architecture (React Native + Expo)**
```
ui/mobile/BoatOwner/
├── app/            # File-based routing screens
├── components/     # Reusable UI components
├── hooks/          # Custom React hooks (React Query)
├── api/fetch/      # API calls using authFetch
├── interfaces/     # TypeScript interfaces
├── constants/      # API routes, query keys, colors
└── context/        # React context (Auth)
```

---

## 🔒 SECURITY & AUTHENTICATION

### **Authentication Requirements:**
- **ALL API endpoints require authentication** (except `/health`)
- **Use JWT tokens** with access/refresh token flow
- **All API calls use `authFetch`** for automatic token handling
- **Validate user ownership** for boat-related resources
- **Use Expo SecureStore** for sensitive data on mobile

### **Security Best Practices:**
- **Validate all inputs** on both client and server
- **Use proper HTTP status codes** for all responses
- **Store secrets in environment variables** NEVER in code
- **Implement proper error handling** with user-friendly messages

---

## 📊 DATABASE CHANGE WORKFLOW

**Follow these steps for any database schema change (add, modify, or remove columns/tables/relations):**

1. **Plan your change:**
   - Decide what you need to add, remove, or modify in the database (e.g., new table, column, relation, index).
   - Review the current `prisma/schema.prisma` and existing migration scripts for context.

2. **Create a migration script:**
   - In `database/migrations/base/`, create a new SQL file named with the next version and a clear description, e.g., `V1.15.0__add_new_table.sql`.
   - Write the SQL statements to apply your change (e.g., `ALTER TABLE`, `CREATE TABLE`, `DROP COLUMN`).
   - If you need to seed or update data, add those statements as well.
   - **Never edit old migration scripts.** Always add a new one.

3. **DO NOT MANUALLY UPDATE `prisma/schema.prisma`** for schema changes. Instead, use the following step to ensure you allow Prisma to update the schema by running their specific commands.

4. **Apply the migration locally:**
   - Run the migration against your local DB:
     ```zsh
     docker compose down -v
     docker volume prune -f
     docker compose up --build -d
     npx prisma db pull
     npx prisma generate
     ```
   - **Verify the DB structure is correct after migration.**

5. **Regenerate the Prisma client:**
   - After updating the schema, always run:
     ```zsh
     cd server
     npx prisma db pull
     npx prisma generate
     ```
   - This ensures your code uses the latest DB types and models.
   - NEVER MANUALLY UPDATE THE `prisma/schema.prisma` file. Always use the Prisma CLI commands to pull the latest schema from the database

6. **Update code and tests:**
   - Update services, controllers, mappers, and interfaces to use the new/changed DB fields.
   - Add or update tests to cover the new DB logic.
   - Update test data in `server/test-utils/test-data/` if needed.

7. **Validate everything:**
   - Run all tests: `npm run test`
   - Lint and format: `npm run lint --fix && npm run format`
   - Start the server: `npm run start`
   - Make sure there are no errors and all tests pass.

8. **Document your change:**
   - Add a note in the relevant migration guide or changelog.
   - If the change affects API output, update Swagger docs and run `npm run swagger-autogen`.

9. **Commit your changes:**
   - Include the migration script, Prisma schema update, code changes, and tests in your commit.
   - Make sure your commit message references the relevant ticket or feature.

---

## 🚀 FEATURE DEVELOPMENT WORKFLOW

### **Phase 1: Planning & Design**
- [ ] **Domain Analysis**: Identify which entities are affected (User, Boat, Logs, Tasks, Expenses)
- [ ] **Database Impact**: Determine if schema changes are needed
- [ ] **API Design**: Plan RESTful endpoints following existing patterns
- [ ] **UI/UX Flow**: Sketch user interaction flow
- [ ] **Security Review**: Identify authentication/authorization requirements

### **Phase 2: Backend Implementation**
- [ ] **Database Migration**: Create Flyway migration if schema changes needed
- [ ] **Prisma Schema**: Update schema using `npx prisma db pull` after migration
- [ ] **TypeScript Interfaces**: Define/update interfaces in `server/interfaces/`
- [ ] **Service Layer**: Implement business logic in `server/services/`
- [ ] **Controller Layer**: Handle HTTP requests in `server/controllers/`
- [ ] **Router Configuration**: Add routes in `server/routers/`
- [ ] **Middleware**: Implement validation/auth middleware
- [ ] **Swagger Documentation**: Document all new endpoints
- [ ] **Unit Tests**: Write comprehensive tests
- [ ] **Integration Tests**: Test full request/response cycle

### **Phase 3: Frontend Implementation**
- [ ] **TypeScript Types**: Create/update interfaces in `ui/mobile/BoatOwner/interfaces/`
- [ ] **API Integration**: Create fetch functions in `ui/mobile/BoatOwner/api/fetch/`
- [ ] **Custom Hooks**: Implement React Query hooks in `ui/mobile/BoatOwner/hooks/`
- [ ] **Components**: Build reusable UI components
- [ ] **Screens**: Create/update screens in `ui/mobile/BoatOwner/app/`
- [ ] **Navigation**: Update routing if needed
- [ ] **State Management**: Implement with React Query + Context
- [ ] **Form Validation**: Use Formik + Yup for forms
- [ ] **Error Handling**: Implement comprehensive error states
- [ ] **Loading States**: Add loading indicators
- [ ] **Testing**: Write unit tests for components and hooks

### **Phase 4: Integration & Deployment**
- [ ] **End-to-End Testing**: Test complete feature flow
- [ ] **Performance Testing**: Ensure no performance regression
- [ ] **Security Testing**: Validate authentication and authorization
- [ ] **Code Review**: Get peer review before merging
- [ ] **Documentation**: Update README and architectural docs
- [ ] **Migration Testing**: Test database migrations in staging

---

## 📝 CODING STANDARDS

### **Backend Standards:**
- **Controllers MUST NOT contain business logic** - only HTTP handling
- **Services MUST handle all business logic** - controllers call services
- **All database access MUST go through Prisma ORM**
- **Every endpoint MUST have authentication** (except `/health`)
- **All inputs MUST be validated** using express-validator
- **All endpoints MUST be documented** in Swagger
- **All functions MUST have TypeScript types**

### **Frontend Standards:**
- **All API calls MUST use authFetch** (automatic token handling)
- **All server state MUST use React Query** - no useState for server data
- **All forms MUST use Formik + Yup** for validation
- **All components MUST be TypeScript** with proper interfaces
- **All screens MUST handle loading/error states**
- **Follow FRONTEND_RULES.md** for styling and structure

---

## 🧪 TESTING REQUIREMENTS

### **Backend Testing:**
- **Unit tests** for ALL services and controllers
- **Integration tests** for ALL API endpoints
- **Mock external dependencies** (database, external APIs)
- **Test error scenarios** not just happy paths
- **Achieve minimum 80% code coverage**

### **Frontend Testing:**
- **Unit tests** for ALL components and hooks
- **Integration tests** for critical user flows
- **Mock API calls** in tests
- **Test error states** and loading states
- **Test accessibility** features

### **Testing Commands:**
```zsh
# Backend tests
cd server && npm test

# Frontend tests
cd ui/mobile/BoatOwner && npm test

# All tests
make test_all
```

---

## 🚫 PROHIBITED PRACTICES

### **Absolutely Forbidden:**
- **❌ Direct database queries** in controllers
- **❌ Business logic** in controllers
- **❌ Hardcoded API URLs** or secrets
- **❌ Using `any` type** in TypeScript
- **❌ Skipping authentication** on new endpoints
- **❌ Not handling errors** properly
- **❌ Not writing tests** for new features
- **❌ Not validating user inputs**
- **❌ Breaking existing functionality**
- **❌ Not updating documentation**

---

## 🔄 DEVELOPMENT ENVIRONMENT SETUP

### **Backend Setup:**
```zsh
cd server
npm install
docker compose up -d  # Start database
npm run start         # Start server
npm run test          # Run tests
npm run swagger-autogen  # Generate Swagger docs
```

### **Frontend Setup (React Native + Expo):**
```zsh
cd ui/mobile/BoatOwner
npm install
npx expo start         # Start Expo development server (required for React Native mobile app)
# Use Expo Go app on your device or an emulator to run the app
npm run test           # Run tests
npm run lint           # Lint code
```

### **Useful Make Commands:**
```zsh
make start_local_db      # Start local database
make start_backend       # Start backend with local DB
make start_backend_docker # Start backend with Docker
make test_all           # Run all tests
```

---

## 📋 DOMAIN-SPECIFIC GUIDELINES

### **User Management:**
- **User authentication** required for all operations
- **Password hashing** with bcrypt
- **JWT token management** with refresh tokens
- **User data validation** on all inputs

### **Boat Management:**
- **User ownership validation** for all boat operations
- **Boat association** required for logs, tasks, expenses
- **Proper foreign key constraints** in database

### **Logs Management:**
- **GPS coordinates** stored as JSON array
- **Crew members** stored as string array
- **Photo URLs** stored as string array
- **Location recording** state management
- **Timestamps** for start/end of logs

### **Tasks Management:**
- **Status tracking** (pending, in-progress, completed)
- **Task assignment** to specific boats
- **Due date management** if implemented
- **Task categories** if implemented

### **Expenses Management:**
- **Decimal precision** for money amounts
- **Expense categories** validation
- **Date validation** for expense dates
- **Currency handling** if multiple currencies

---

## 📞 GETTING HELP

### **When Stuck:**
1. **Check existing code** in the same domain
2. **Review architectural documentation**
3. **Search GitHub issues** for similar problems
4. **Ask team lead** for guidance
5. **Create detailed GitHub issue** for complex problems

### **Key Resources:**
- **Project Documentation**: All `.md` files in root and `/doc/` directory
- **Trello Board**: https://trello.com/b/7GH6OZzf/boatowner
- **Architecture Docs**: `/doc/architecture/` directory
- **Frontend Rules**: `/ui/mobile/BoatOwner/FRONTEND_RULES.md`
- **Development Rules**: `/DEVELOPMENT_RULES.md`

---

## 🎯 COMMIT MESSAGE FORMAT

```
feat(domain): brief description

- Detailed explanation of changes
- Why this change was made
- Any breaking changes
- Testing performed

Closes #123
```

---

## 🚢 DEPLOYMENT NOTES

### **CI/CD Pipeline:**
- **GitHub Actions** handle automatic deployment
- **Database migrations** deploy automatically on master
- **Backend deploys** to staging first, then production
- **Frontend builds** and deploys through Expo

### **Environment Variables:**
- **Backend**: Configure database, JWT secrets, API keys
- **Frontend**: Configure API URLs, Google Maps API key
- **Production**: Use proper HTTPS URLs and secure secrets

---

**Remember: These instructions ensure code quality, security, and maintainability. Following them protects both the project and your fellow developers.**

**🚢 Built with ❤️ for boat owners**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/conorwaltersUno)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/conorwaltersUno)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
