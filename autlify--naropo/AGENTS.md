Provide project context and coding guidelines that AI should follow when generating code, answering questions, or reviewing changes.

# Autlify
Autlify is a platform that enables users to create and manage agencies, clients, and projects with features like authentication, passkey management, and file uploads. The codebase primarily uses TypeScript and React, with a focus on security and user experience.

## Instructions Purpose
These instructions are designed to guide AI in generating, reviewing, and analyzing code within the Autlify project. They ensure that all contributions align with the project's architecture, coding standards, reusability, and best practices.

## Important Guidelines
1. Always prioritize code reuse by searching the existing codebase before creating new files or components.
2. Follow established coding conventions and patterns used throughout the project.
3. Ensure all new code is well-documented with clear comments explaining complex logic with sample usages and expected inputs/outputs.
4. Maintain a strong focus on security, especially in authentication and data handling.
5. Adhere to performance, scalability, flexibility, dynamic best practices.
6. Ensure all new features include permission keys, entitlement checks, and appropriate tests (unit, integration, end-to-end) to maintain code quality.
7. When generating code, ensure it integrates seamlessly with the existing architecture and follows established patterns.
8. When reviewing code, focus on correctness, completeness, functionality, efficiency, readability, relatedness and adherence to project guidelines.
9. When responding to queries about the project, provide context-aware answers that reflect the current architecture and design principles.
10. When generating documentationas or explanations directly kept them in `/site/docs/` folder and follow the existing documentation style and separate files for different audiences:
    - Developer Guides
    - User Manuals
    - API References
    - Internal Docs

## Single Source of Truth (SSoT) Resources
- **Registry**: The primary source for all mappings, resolvers, and definitions is the Registry located at `src/lib/registry/`. It includes:
    `KEYS`: IDENTIFIERS in a hierarchical structure with pattern `${module}`.`Submodule`.`${Feature}`.`${Action}` 
       |
       V
    `Feature`: The main functionality or module (e.g., `${module}`.`Submodule`.`${Feature}`) as type `FeatureKey`
       |
       V
    `Entitlement`: The entitlements of a specific plan that grants access to features
       |
       V
    `Plans`: The subscription plans or add-ons available in the system
       |
       V
    `Permission`: The required permissions `key` to access features `${module}`.`Submodule`.`${Feature}`.`${Action}` as type `PermissionKey` aka `ActionKey`
       |
       V
    `Role`: The system/custom roles with associated permissions `keys` within the system
       |
       V
    `Action`: The specific operations that can be performed on features `${module}`.`Submodule`.`${Feature}`.`${Action}` as type `ActionKey` aka `PermissionKey`
       |
       V
    `API Endpoint`: The corresponding API routes that implement the feature actions and enforce  `/api/${module}/Submodule/${Feature}`
       |
       V
     `Tracking`: The tracking and analytics events related to feature usage and actions  `${module}`.`Submodule`.`${Feature}` and `${Action}`




## Project Overview
- **Authentication**: Utilizes NextAuth.js for secure user authentication, including passkey support.
- **File Uploads**: Implements UploadThing for handling file uploads efficiently.
- **Database**: Uses Prisma ORM to interact with a PostgreSQL database.
- **Styling**: Employs Tailwind CSS along with Aceternity UI and Re UI for consistent and responsive design.
- **Testing**: Incorporates Playwright for end-to-end testing.



## General Guidelines
- Follow TypeScript best practices, ensuring type safety and clarity.
- Adhere to React conventions, using functional components and hooks.
- Maintain consistent code formatting and style throughout the project.
- Prioritize security, especially in authentication and data handling.
- Write clear and concise comments to explain complex logic.
- Always validate and sanitize user inputs to prevent security vulnerabilities.

## Important Notes
- Ensure all API routes handle errors gracefully and provide meaningful feedback to the client.
- Ensure `#codebase` is utilized effectively; avoid reinventing the wheel by reusing existing utilities and components.
- When actioning tasks or requests (eg.,`generate code`, `review code`, `search codebase`, `analyze code` and etc), always execute part by part to avoid server errors 504 (Gateway Timeout) due to long processing times.
- When responding to queries about the project, provide context-aware answers that reflect the current architecture and design principles.
- When generating documentation or explanations, focus on clarity and conciseness.
- When summarizing or analyzing code, do not generate markdown documentation unless explicitly requested.
- Adhere to existing code styles, conventions, best practices, reuse available code in the `#codebase` before creating new implementations.
- When generating code, ensure it integrates seamlessly with the existing architecture and follows established patterns.
- When reviewing code, focus on correctness, efficiency, readability, and adherence to project guidelines.


## Requirements
- All new features must include appropriate unit and integration tests.
- Ensure compatibility with the latest versions of dependencies listed in the tech stack.
- Never set or estimate timelines for tasks / feature / implementations, focus on speed, quality and completeness.
- Even if not explicitly mentioned, always consider edge cases, potential failure points, and codebase searching before your implementations.
- Code should be optimized for performance and scalability where applicable.
- Never compromise on the following principles:

    - ✅ Code quality over speed  
    - 🔒 Security over usability  
    - 🔧 Maintainability over clever tricks  
    - ♿ Accessibility over flashy design  
    - 🔄 Flexibility over simplicity  
    - 📈 Scalability over cost-cutting  
    - ⚡ Reliability over shiny new features  
    - 🎨 Customizability over ease-of-use shortcuts  
    - 🔗 Integratability over isolation  
    - 📚 Documentation over brevity  
    - 🛠 Functionality over aesthetics  
    - 🔌 Compatibility over innovation   
    

## UI/UX Guidelines
- When working on UI components, ensure they align with the existing design system.
- Refer to `aceternity ui` and `re ui`, dark mode is a must and should be consistently applied across all components.
- Ensure all components are responsive and accessible across different devices, screen sizes and theme modes.
- When dealing with authentication and sensitive data, ensure compliance with relevant security standards and protocols.
- Test all new features thoroughly, including edge cases, to ensure reliability and robustness.


## Our Tech Stack
- **Frontend**: React React-DOM v19.1+, TypeScript v5.0+, Next.js v16.1+
- **Backend**: Node.js v22.21+, Next.js API Routes v16.1+, Prisma ORM v7.2+
- **Database**: PostgreSQL v15+
- **Authentication**: NextAuth.js v5.0+, Passkeys (WebAuthn)
- **File Uploads**: UploadThing v7.0+
- **Styling**: Tailwind CSS v4.1.6+, Aceternity UI, Re UI
- **Testing**: Jest v29+, React Testing Library v14+


## Our Vision
To create a seamless platform for managing agencies, clients, and projects with a focus on security, flexibility, usability, scalability and dynamically adapting to user needs. We aim to empower users with robust tools while maintaining an intuitive and accessible user experience.


## Specific Areas of Focus
- **Passkey Management**: Ensure seamless integration of passkey registration, authentication, and error handling.
- **API Development**: Follow RESTful principles and ensure all endpoints are well-documented and tested.
- **User Experience**: Prioritize intuitive navigation and user-friendly interfaces in all components.
- **State Management**: Use React's built-in state management effectively; consider context or other libraries only when necessary.


## Code Reuse and File Management

### Before Creating New Files:
1. **MUST search codebase first** using `semantic_search` or `grep_search`
2. Check for existing similar functionality, utilities, or components
3. Reuse existing code; only create new files if genuinely needed
4. If similar code exists, refactor/extend it instead of duplicating

### File Creation Rules:
- ❌ **NEVER** create duplicate utilities (check `src/lib/` first)
- ❌ **NEVER** create new components if similar exists (check `src/components/`)
- ❌ **NEVER** create new API routes with duplicate logic
- ✅ **ALWAYS** check existing patterns before creating new files
- ✅ **ALWAYS** follow existing folder structure

### Folder Structure Compliance:
- API routes: `src/app/api/[module]/[resource]/route.ts`
- Components: `src/components/[category]/[component-name].tsx`
- Libraries: `src/lib/[module]/[functionality].ts`
- Types: `src/types/[module].ts` (not scattered)
- Hooks: `src/hooks/use-[functionality].ts`

### Required Checks Before File Creation:
1. Run `semantic_search` for similar functionality
2. Check existing exports in target directory
3. Verify no duplicate logic in `src/lib/utils` or `src/lib/helpers`
4. If extending existing file, use `replace_string_in_file` instead

### Examples:
❌ Creating `src/lib/format-date.ts` when `src/lib/utils.ts` has `formatDate()`
✅ Adding to existing `src/lib/utils.ts`

❌ Creating `src/components/ui/custom-button.tsx` when `src/components/ui/button.tsx` exists
✅ Extending existing button component

❌ Creating new validation in `src/lib/validators/user.ts` when similar exists
✅ Reusing existing validation logic

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Autlify)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Autlify)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
