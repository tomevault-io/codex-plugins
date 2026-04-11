<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Citizen Feedback Platform - Copilot Instructions

## Project Overview
This is a modern citizen feedback and engagement platform built with Next.js frontend and Express.js backend. The platform allows citizens to submit feedback, suggestions, and complaints to local authorities, and enables authorities to respond and manage these submissions.

## Tech Stack

### Frontend
- **Framework**: Next.js 14 with App Router
- **Language**: TypeScript
- **Styling**: Tailwind CSS with custom design system
- **Components**: Custom components with Lucide React icons
- **Maps**: Leaflet.js for location mapping
- **Forms**: React Hook Form with Yup validation
- **HTTP Client**: Axios
- **Notifications**: React Hot Toast

### Backend
- **Framework**: Express.js with TypeScript
- **Database**: PostgreSQL with Prisma ORM
- **Authentication**: JWT tokens
- **File Upload**: Cloudinary integration
- **Email**: Nodemailer
- **SMS**: Twilio
- **Security**: Helmet, CORS, Rate limiting

### Deployment
- **Containerization**: Docker with docker-compose
- **Reverse Proxy**: Nginx
- **Database**: PostgreSQL container
- **Caching**: Redis container

## Code Guidelines

### Frontend
- Use TypeScript for all components and utilities
- Follow React functional components with hooks
- Use Tailwind CSS utility classes with custom component classes
- Implement responsive design (mobile-first approach)
- Use proper error handling and loading states
- Follow the established color scheme and design tokens

### Backend
- Use TypeScript for all server code
- Implement proper error handling with custom error classes
- Use Prisma for database operations
- Follow RESTful API conventions
- Include proper authentication and authorization
- Implement input validation using Joi or similar
- Use proper HTTP status codes

### Database Schema
- Use Prisma schema for type-safe database operations
- Follow proper relationship definitions
- Include proper indexes for performance
- Use enums for fixed value sets

### Security
- Implement authentication for protected routes
- Use role-based access control (RBAC)
- Sanitize all inputs
- Use HTTPS in production
- Implement rate limiting
- Validate file uploads

## Key Features to Implement
1. **Feedback Submission**: Anonymous and authenticated feedback with location mapping
2. **Admin Dashboard**: Management interface for authorities
3. **Real-time Notifications**: Email/SMS updates for status changes
4. **Multilingual Support**: Arabic and English language support
5. **Mobile Optimization**: PWA capabilities
6. **Image Upload**: Drag-and-drop with Cloudinary
7. **Analytics**: Statistics and reporting dashboard
8. **Role Management**: Multiple user roles and permissions

## Development Practices
- Use semantic commit messages
- Implement proper error boundaries
- Write unit tests for critical functionality
- Use environment variables for configuration
- Follow accessibility best practices (WCAG guidelines)
- Implement proper logging for debugging
- Use TypeScript strict mode
- Follow ESLint and Prettier configurations

## API Conventions
- Use RESTful endpoints with proper HTTP methods
- Return consistent JSON response format
- Include pagination for list endpoints
- Implement proper error response structure
- Use status codes appropriately
- Include request/response TypeScript interfaces

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/alsaidisoft)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/alsaidisoft)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
