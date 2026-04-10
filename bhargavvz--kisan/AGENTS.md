<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Project Kisan Development Instructions

## Project Overview
This is a Flutter app called "Project Kisan" designed for farmers in India. It provides crop disease diagnosis, market price information, and government subsidy details.

## Architecture Guidelines
- Follow clean architecture principles with proper separation of concerns
- Use Provider pattern for state management
- Implement responsive design for various screen sizes
- Follow Material Design 3 guidelines

## Code Standards
- Use meaningful variable and function names
- Add comments for complex business logic
- Follow Dart/Flutter naming conventions
- Keep functions small and focused on single responsibility
- Use const constructors where possible for performance

## Design System
- Primary Color: #2D7A46 (Green)
- Accent Color: #F8B133 (Orange)
- Background: #F2F7F1 (Light Green)
- Use Poppins font family
- Maintain 48px minimum touch targets for accessibility
- Follow consistent spacing: 4px, 8px, 16px, 24px, 32px, 48px

## Localization
- Support English and Kannada languages
- Use context.t() for all user-facing strings
- Add new translations to both en.json and kn.json files
- Test UI with both languages to ensure proper layout

## State Management
- Use Provider for application state
- Keep providers focused and single-purpose
- Avoid complex state mutations
- Use proper error handling and loading states

## UI Guidelines
- Use custom widgets from the widgets/ folder
- Implement proper loading and error states
- Ensure accessibility compliance
- Test on different screen sizes
- Use semantic colors (success, error, warning, info)

## File Organization
- Models: Data structures and business entities
- Providers: State management classes
- Screens: Full-screen UI components
- Widgets: Reusable UI components
- Utils: Helper functions, constants, and configurations

## Testing Considerations
- Write unit tests for business logic
- Test state management thoroughly
- Verify localization works correctly
- Test accessibility features
- Ensure responsive design works on various devices

## Future Backend Integration
- Design APIs to be RESTful
- Plan for offline-first architecture
- Consider data synchronization strategies
- Implement proper error handling for network requests
- Plan for user authentication and authorization

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Bhargavvz)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Bhargavvz)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
