<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# DiaMod2R - Copilot Instructions

This is a web application for managing Diablo 2 mod files, built with Node.js/TypeScript backend and React/TypeScript frontend.

## Project Structure

- **Backend**: Node.js + Express + TypeORM + PostgreSQL
- **Frontend**: React + TypeScript + Axios
- **Database**: PostgreSQL with models for Mods and CharStats

## Key Guidelines

1. **TypeScript**: Always use strict typing and interfaces
2. **Error Handling**: Implement comprehensive error handling in all services
3. **Database**: Use TypeORM decorators and repositories pattern
4. **API**: Follow RESTful conventions for all endpoints
5. **React**: Use functional components with hooks
6. **File Processing**: Handle large files efficiently and validate data thoroughly

## Code Style

- Use camelCase for variables and functions
- Use PascalCase for classes and components
- Use kebab-case for file names
- Prefer async/await over promises
- Always validate user inputs
- Add comprehensive error messages

## Security Considerations

- Validate all file paths to prevent directory traversal
- Sanitize database inputs
- Handle file system errors gracefully
- Use environment variables for sensitive configuration

## Performance

- Implement pagination for large datasets
- Use streaming for file operations when possible
- Optimize database queries with appropriate indexes
- Cache frequently accessed data when appropriate

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CareZapato) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
