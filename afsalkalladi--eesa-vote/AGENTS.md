# Copilot Instructions for Django Election Voting System

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

## Project Overview

This is a Django-based Class Election Voting System with the following key features:

- Token-based voter authentication (UUID tokens)
- Multiple position elections with time-limited voting
- Anonymous voting with audit trail capabilities
- Live and final results display
- Admin tools for managing elections, voters, and candidates
- Bootstrap 5 responsive frontend

## Code Style Guidelines

- Follow Django best practices and PEP 8
- Use descriptive variable and function names
- Add comprehensive docstrings for all models, views, and functions
- Implement proper error handling and validation
- Use Django's built-in security features
- Maintain separation of concerns between models, views, and templates

## Security Considerations

- Implement proper CSRF protection
- Use Django's authentication and authorization system
- Validate all user inputs
- Implement rate limiting for sensitive operations
- Ensure audit trails are only accessible to authorized users
- Use secure token generation for voter authentication

## Database Design

- Use Django ORM with proper relationships
- Implement appropriate indexes for performance
- Use timezone-aware datetime fields
- Ensure data integrity with proper constraints

## Frontend Guidelines

- Use Bootstrap 5 for responsive design
- Implement progressive enhancement
- Add proper form validation
- Include accessibility features
- Use semantic HTML structure

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Afsalkalladi) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
