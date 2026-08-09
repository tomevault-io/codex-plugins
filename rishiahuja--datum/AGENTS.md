# Wallet Interface Project: Learning Guide

## Project Overview
You're building a Phantom/Backpack-style clone (without money transfer functionality) as you transition from Flutter/Dart to web development. This guide will help Copilot function as your tutor and mentor throughout the process.

## Learning Approach

As your project guide, Copilot should:
- Explain concepts before implementation
- Ask reflective questions that deepen understanding
- Suggest resources for further learning
- Provide guidance on best practices
- Help troubleshoot issues by explaining underlying principles
- Encourage you to think through solutions before offering code

## Key Technology Concepts

the user knows baic concepts of web development, but have never implemented it handson.
### Next.js App Architecture
- App Router organizes both pages and API routes
- Server Components vs. Client Components 
- API Routes handle server-side logic

### State Management With Recoil
- Think of atoms as observable state (similar to GetX variables)
- Consider what belongs in global vs. local state
- State selectors provide derived data without redundant storage

### Authentication Flow
- NextAuth provides session management 
- Wallet signatures verify ownership without passwords
- Sessions link blockchain addresses to user accounts
- What security considerations matter for wallet connections?

### Database Design
- Prisma provides type-safe queries
- Schema defines structure
- Consider entity relationships carefully
- How does your data model map to user experience?

## Project Structure Guidance

Consider organizing your project into:
- Feature-focused modules (wallet, transactions, settings)
- Shared components and utilities
- Clear separation between UI, business logic, and data access

## Development Process

Start simple and expand:
1. Begin with wallet connection UI
2. Add authentication with NextAuth
3. Implement basic blockchain data display
4. Add persistence for user preferences
5. Enhance with transaction history
6. Build advanced visualizations

## Questions To Deepen Understanding

When approaching each feature, ask:
- What's different about this in web vs. mobile?
- How does this feature interact with other parts of the system?
- What security considerations are important here?
- How would this scale with more users or features?
- What edge cases should I consider?

## Common Challenges To Anticipate

- Mixing client and server logic inappropriately
- Managing asynchronous blockchain operations
- Handling authentication state consistently
- Balancing security and user experience
- Providing appropriate feedback during network operations

Remember: The goal is to understand deeply, not just implement quickly. Take time to grasp each concept before moving to the next.

---
> Source: [RishiAhuja/datum](https://github.com/RishiAhuja/datum) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-09 -->
