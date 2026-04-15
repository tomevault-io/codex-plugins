# Copilot Instructions

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

This is a medical patient management application for Chronic Kidney Disease (CKD) patients. The project uses:

- **Frontend**: Next.js 14 with TypeScript and Tailwind CSS
- **Backend**: Firebase (Firestore, Auth, Cloud Functions, Cloud Messaging)
- **PDF Generation**: pdf-lib and jsPDF
- **State Management**: React Context API

## Key Features:
1. **Authentication**: Firebase Auth with role-based access (doctor, nurse)
2. **Patient Management**: Full CRUD operations for patients with medical history
3. **Smart Alerts**: Automatic detection of critical clinical values with thresholds
4. **PDF Generation**: Complete patient file export with medical data
5. **Notifications**: Push notifications via Firebase Cloud Messaging

## Code Standards:
- Use TypeScript for all components and services
- Follow Next.js 14 App Router conventions
- Implement responsive design with Tailwind CSS
- Use Firebase v9+ modular SDK
- Include proper error handling and loading states
- Follow medical data privacy and security best practices

## Project Structure:
- `src/app/` - Next.js pages and layouts
- `src/components/` - Reusable UI components
- `src/context/` - React Context for state management
- `src/services/` - Firebase services and API calls
- `src/utils/` - Utility functions including PDF generation
- `src/types/` - TypeScript type definitions

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Hec8) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
