# GitHub Copilot Instructions - Nginx WAF Management Platform

## 🎯 Project Overview

This is a comprehensive Nginx management system with ModSecurity WAF, providing domain management, SSL certificates, and real-time monitoring. The project is built as a monorepo using:

- **Backend**: Node.js + Express + TypeScript + Prisma + PostgreSQL
- **Frontend**: React + TypeScript + Vite + TanStack Router + ShadCN UI + Tailwind CSS
- **Infrastructure**: Docker + Nginx + ModSecurity WAF
- **Tooling**: Turbo monorepo, pnpm, Vitest

## 📁 Architecture & Structure

### Monorepo Layout
```
/apps/api/          # Backend API server (Express + Prisma)
/apps/web/          # Frontend React application
/apps/docs/         # Documentation site
/docker/            # Docker configurations
/config/            # Nginx configurations
/scripts/           # Deployment and setup scripts
```

### Backend Domain-Driven Design (DDD)
```
/apps/api/src/
├── domains/        # Business domain modules
│   ├── auth/       # Authentication & authorization
│   ├── users/      # User management
│   ├── domains/    # Domain/site management
│   ├── ssl/        # SSL certificate management
│   ├── modsec/     # ModSecurity WAF configuration
│   ├── acl/        # Access Control Lists
│   ├── alerts/     # Alert system
│   ├── logs/       # Log management
│   ├── performance/ # Performance monitoring
│   └── dashboard/  # Dashboard data
├── middleware/     # Express middleware
├── routes/         # API route definitions
├── utils/          # Utility functions
├── shared/         # Shared types & constants
└── config/         # Application configuration
```

## 🛡️ Code Review Guidelines

### Security Focus
Given this is a security-focused WAF management platform, prioritize:

1. **Authentication & Authorization**
   - Review JWT token handling and validation
   - Check role-based access control (RBAC) implementation
   - Validate input sanitization and XSS Attack, SQL injection prevention
   - Ensure secure password handling with bcrypt

2. **Input Validation**
   - All user inputs must be validated using express-validator
   - File uploads should have strict type and size limitations
   - API endpoints should validate request bodies and parameters

3. **Configuration Security**
   - Nginx configuration changes should be validated
   - ModSecurity rules should be tested before deployment
   - SSL certificate handling must be secure

### Backend Code Standards

#### API Design
- Follow RESTful principles
- Use consistent HTTP status codes
- Implement proper error handling with structured error responses
- Use middleware for cross-cutting concerns (auth, validation, logging)

#### Database & Prisma
- Use Prisma schema best practices with proper relations
- Implement database transactions for multi-step operations
- Use proper indexing for performance
- Follow migration naming conventions

#### TypeScript Standards
- Use strict TypeScript configuration
- Define proper interfaces and types
- Avoid `any` types - use proper type definitions
- Implement generic types where appropriate

### Frontend Code Standards

#### React & TypeScript
- Use functional components with hooks
- Implement proper TypeScript interfaces for props and state
- Use React Query (TanStack Query) for server state management
- Follow component composition patterns

#### Routing & Navigation
- Use TanStack Router for type-safe routing
- Implement route-based code splitting
- Handle loading and error states properly

#### UI/UX Guidelines
- Use ShadCN UI components consistently
- Follow Tailwind CSS utility-first approach
- Implement responsive design patterns
- Ensure accessibility (a11y) standards

### Performance & Monitoring

#### Backend Performance
- Monitor API response times
- Implement proper caching strategies
- Use database query optimization
- Handle concurrent requests properly

#### Frontend Performance
- Implement lazy loading for routes and components
- Optimize bundle size with proper code splitting
- Use React.memo and useMemo for expensive operations
- Monitor Core Web Vitals

## 🧪 Testing Standards

### Backend Testing (Vitest)
- Unit tests for service layer functions
- Integration tests for API endpoints
- Mock external dependencies (file system, nginx configs)
- Test error handling scenarios

### Test File Organization
```
/apps/api/src/
├── domains/auth/__tests__/
├── domains/users/__tests__/
└── utils/__tests__/
```

## 🔧 Development Workflow

### Branch Naming
- `feature/` for new features
- `fix/` for bug fixes
- `security/` for security-related changes
- `refactor/` for code refactoring

### Commit Messages
Follow conventional commits:
- `feat:` for new features
- `fix:` for bug fixes
- `security:` for security improvements
- `refactor:` for code refactoring
- `test:` for adding tests
- `docs:` for documentation updates

## 📝 Specific Review Checkpoints

### When Reviewing Authentication Code
- [ ] JWT tokens are properly validated and expired
- [ ] Passwords are hashed using bcrypt with proper salt rounds
- [ ] Session management is secure
- [ ] Two-factor authentication implementation is correct
- [ ] Rate limiting is implemented for login attempts

### When Reviewing API Endpoints
- [ ] Input validation is comprehensive
- [ ] Error responses don't leak sensitive information
- [ ] Proper HTTP status codes are used
- [ ] Authentication middleware is applied where needed
- [ ] Database queries are optimized and safe

### When Reviewing Frontend Components
- [ ] TypeScript types are properly defined
- [ ] Loading and error states are handled
- [ ] Forms use proper validation
- [ ] Sensitive data is not logged or exposed
- [ ] Accessibility attributes are included

### When Reviewing Infrastructure Code
- [ ] Docker configurations are secure and optimized
- [ ] Nginx configurations follow security best practices
- [ ] ModSecurity rules are properly formatted
- [ ] SSL/TLS configurations are secure
- [ ] Environment variables are properly handled

### When Reviewing Database Schemas
- [ ] Proper constraints and indexes are defined
- [ ] Sensitive data fields are appropriately handled
- [ ] Migration scripts are safe and reversible
- [ ] Foreign key relationships are correctly defined

## 🚨 Security Red Flags

Immediately flag these security concerns:
- Hardcoded secrets or passwords
- SQL injection vulnerabilities
- Missing input validation
- Insecure file upload handling
- Improper authentication checks
- Sensitive data in logs
- Missing CORS configuration
- Weak cryptographic practices

## 🎯 Domain-Specific Knowledge

### Nginx & ModSecurity
- Understand nginx configuration syntax
- Review ModSecurity rule format and OWASP CRS
- Validate SSL certificate handling
- Check load balancing configurations

### WAF Management
- Review firewall rule logic
- Validate IP whitelist/blacklist functionality
- Check GeoIP filtering implementation
- Ensure proper logging of security events

## 📊 Performance Monitoring

Always consider:
- Database query performance and N+1 problems
- API response time optimization
- Frontend bundle size and loading performance
- Memory usage and potential leaks
- Proper caching implementation

This platform manages critical security infrastructure, so thorough code review is essential for maintaining system security and reliability.

---
> Source: [TinyActive/nginx-love](https://github.com/TinyActive/nginx-love) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-03 -->
