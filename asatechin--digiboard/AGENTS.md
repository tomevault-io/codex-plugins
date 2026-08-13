<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# DigiBoard Project Instructions

This is a full-stack DigiBoard application with the following architecture:

## Backend (Node.js + Express + MongoDB)
- Located in `/backend` directory
- Uses Express.js for REST API
- MongoDB with Mongoose for data persistence
- Models: Teacher, Lecture
- API endpoints for schedule management, lectures, and teachers

## Frontend (Flutter)
- Located in `/frontend` directory
- Flutter mobile/web application
- Shows lecture schedules and teacher information
- Modern Material Design 3 UI
- Real-time data from backend API

## Key Features
- Display next upcoming lecture with teacher details
- Today's schedule view
- Weekly schedule overview
- Teacher profile information with contact details
- Responsive and modern UI design

## Development Guidelines
- Backend runs on port 5000
- Frontend connects to http://localhost:5000/api
- Use consistent error handling and loading states
- Follow Material Design principles for UI
- Maintain separation between models, services, and UI components

## API Endpoints
- GET /api/schedule/next - Get next upcoming lecture
- GET /api/schedule/today - Get today's lectures
- GET /api/schedule/week - Get weekly schedule
- GET /api/lectures - Get all lectures
- GET /api/teachers - Get all teachers

When working on this project, ensure proper error handling, loading states, and maintain the existing code structure and patterns.

---
> Source: [ASAtechin/digiboard](https://github.com/ASAtechin/digiboard) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-13 -->
