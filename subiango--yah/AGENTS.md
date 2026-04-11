# yah Development Guidelines

Auto-generated from all feature plans. Last updated: 2025-09-26

## Constitutional Principles (NON-NEGOTIABLE)
- **Security-First**: Cryptographically secure access codes, one-time use enforcement, server-side validation
- **Test-Driven Development**: Write tests first, ensure they fail, then implement
- **UX Consistency**: Bootstrap theming, immediate form feedback, mobile-responsive design
- **Performance Standards**: <500ms form submission, <2s QR generation, async email delivery
- **Code Quality**: Single responsibility, clear naming, externalized config, documented schemas

## Active Technologies
- JavaScript (Node.js 18+), HTML5, CSS3, ES6+ + Express.js, MongoDB driver, Nodemailer, qrcode npm package (002-youth-event-registration)

## Project Structure
```
backend/
frontend/
tests/
```

## Security Requirements
- Access code generation MUST use crypto.randomBytes or equivalent
- Database operations MUST use transactions for one-time code enforcement  
- All inputs MUST be validated and sanitized server-side
- Email services MUST use TLS encryption

## Performance Targets
- Form submission: <500ms response time
- QR code generation: <2 seconds completion
- Admin dashboard: Support 1000+ registrations
- Database queries: Proper indexing on access codes and emails

## Commands
npm test [ONLY COMMANDS FOR ACTIVE TECHNOLOGIES][ONLY COMMANDS FOR ACTIVE TECHNOLOGIES] npm run lint

## Code Style
JavaScript (Node.js 18+), HTML5, CSS3, ES6+: Follow standard conventions

## Recent Changes
- 002-youth-event-registration: Added JavaScript (Node.js 18+), HTML5, CSS3, ES6+ + Express.js, MongoDB driver, Nodemailer, qrcode npm package

<!-- MANUAL ADDITIONS START -->
<!-- MANUAL ADDITIONS END -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/SUBiango)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/SUBiango)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
