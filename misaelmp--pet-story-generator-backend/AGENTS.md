<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Pet Story Generator Backend - Copilot Instructions

## Project Overview

This is a Node.js/TypeScript backend service for generating personalized pet stories using OpenAI's GPT API. The service includes integrations with PIMS (Pet Information Management System) and Xano database.

## Architecture Guidelines

- Use TypeScript with strict type checking
- Follow Express.js patterns with controllers, services, and middleware
- Implement proper error handling with custom error types
- Use Zod for request validation
- Follow RESTful API conventions

## Code Style

- Use async/await instead of promises
- Implement proper TypeScript interfaces for all data structures
- Use dependency injection where appropriate
- Follow the existing project structure (controllers, services, middleware, routes, types)
- Include proper error handling in all functions

## Security Considerations

- Always validate user input using Zod schemas
- Implement rate limiting for API endpoints
- Use environment variables for sensitive configuration
- Sanitize user inputs before processing
- Follow CORS best practices

## OpenAI Integration

- Use the official OpenAI SDK
- Implement proper error handling for API failures
- Use appropriate temperature settings for creative text generation
- Implement token limit controls based on story length
- Cache responses when appropriate

## Testing

- Write unit tests for services and controllers
- Mock external API calls in tests
- Test error scenarios and edge cases
- Validate TypeScript types in tests

## Environment Setup

- Use dotenv for environment variable management
- Provide clear .env.example with all required variables
- Document all environment variables in README.md
- Use different configurations for development and production

---
> Source: [MisaelMP/pet-story-generator-backend](https://github.com/MisaelMP/pet-story-generator-backend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-01 -->
