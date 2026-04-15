# Toy Shop Billing System - Project Guide

## Project Overview
Desktop billing system for toy shops built with Electron, React, Node.js, and SQLite.

## Tech Stack
- **Frontend**: React 18, React Router, CSS3
- **Backend**: Node.js, Express.js
- **Database**: SQLite3
- **Desktop**: Electron
- **Authentication**: JWT + bcrypt

## Project Structure
```
d:/Toy/
├── backend/          # Node.js backend (port 5000)
├── frontend/         # React app (port 3000)
├── .vscode/          # VS Code tasks
├── QUICKSTART.md     # Quick start guide
├── SETUP.md          # Detailed setup guide
└── README.md         # Full documentation
```

## Quick Commands
- `npm run dev` - Start frontend + backend
- `npm run backend` - Start backend only
- `npm run electron-dev` - Launch Electron app
- `npm run electron-build` - Build for production

## Default Credentials
- Username: `admin`
- Password: `admin123`

## Key Features
✅ Admin user/product/inventory management
✅ Cashier billing and receipt generation
✅ Sales reporting and analytics
✅ JWT authentication with role-based access
✅ SQLite database with auto-initialization
✅ Responsive React UI

## Completed Tasks
- [x] Project structure and directory setup
- [x] Backend with Express and SQLite
- [x] Frontend with React and routing
- [x] Authentication system (JWT + bcrypt)
- [x] Admin dashboard components
- [x] Cashier billing interface
- [x] API endpoints (auth, products, sales, inventory)
- [x] Database schema and initialization
- [x] Dependencies installation
- [x] Documentation (README, SETUP, QUICKSTART)

## Database
- Auto-created at: `backend/data/toyshop.db`
- Initializes on first backend startup
- Default admin user created automatically

## Development Notes
- Backend auto-restarts with nodemon on changes
- Frontend auto-refreshes with React dev server
- All routes protected with JWT authentication
- Admin-only endpoints verified with role checks

## Support & Documentation
- QUICKSTART.md - Start in 2 steps
- SETUP.md - Detailed configuration
- README.md - Full feature documentation

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/mrayandar) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
