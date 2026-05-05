# Copilot Instructions for FinAssist

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

## Project Overview
This is a comprehensive financial management platform built with Next.js 14, TypeScript, and modern web technologies. The platform helps users manage their finances, track investments, and receive AI-powered recommendations.

## Tech Stack
- **Frontend**: Next.js 14, React 18, TypeScript, Tailwind CSS
- **UI Components**: ShadCN UI, Lucide React icons
- **Charts**: Recharts
- **Forms**: React Hook Form with Zod validation
- **Authentication**: NextAuth.js with Google OAuth
- **Database**: PostgreSQL with Prisma ORM
- **Backend Scripts**: Python for news fetching and ML
- **State Management**: React Context API, useState, useEffect

## Key Features
1. User authentication with session storage
2. Comprehensive financial dashboard
3. Portfolio tracking with charts and analytics
4. Real-time financial news integration
5. Risk assessment questionnaire
6. AI-powered investment recommendations
7. Notes and financial planning tools

## Code Style Guidelines
- Use TypeScript for all components and utilities
- Follow React best practices with hooks
- Use ShadCN UI components for consistent design
- Implement proper error handling and loading states
- Use Tailwind CSS for styling
- Follow the app router structure in Next.js 14
- Use Prisma for database operations
- Implement proper form validation with Zod

## Database Design
- User profiles with authentication
- Financial data (income, expenses, assets, investments)
- News articles with sentiment analysis
- Risk profiles and recommendations
- Portfolio tracking data

## Security Considerations
- Implement proper authentication and authorization
- Validate all user inputs
- Use environment variables for sensitive data
- Follow OWASP security guidelines
- Implement rate limiting for API endpoints

## Performance Optimization
- Use Next.js server components where possible
- Implement proper caching strategies
- Optimize images and assets
- Use lazy loading for heavy components
- Implement pagination for large datasets

When generating code, prioritize security, performance, and user experience. Always include proper TypeScript types and error handling.

---
> Source: [Julian-Idl/FinAssist](https://github.com/Julian-Idl/FinAssist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-04 -->
