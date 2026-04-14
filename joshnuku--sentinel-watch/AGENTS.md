# Copilot Instructions for Project ORION (sentinel-watch)

## Big Picture Architecture

- **Frontend**: Vite + React + TypeScript (src/)
  - Dashboard for real-time monitoring, alert management, and device status
  - Communicates with backend via REST API and WebSockets
- **Backend**: Express.js + TypeScript (backend/src/)
  - Receives alerts from Raspberry Pi sentinels
  - Persists data in MongoDB
  - Emits real-time updates to dashboard via Socket.io
  - Integrates with Huawei Cloud SMN for SMS alerts
- **Raspberry Pi Client**: Python scripts (backend/examples/raspberry_pi_client.py, backend/raspberry-pi/sentinel_main.py)
  - Sends alerts to backend

## Developer Workflows

- **Frontend**
  - Start dev server: `npm run dev` (root)
  - Build: `npm run build`
  - Uses shadcn-ui for UI components
- **Backend**
  - Start dev server: `cd backend && npm run dev`
  - Build: `npm run build`
  - Production: `npm start`
  - Lint: `npm run lint` / `npm run lint:fix`
  - Environment config: copy `backend/env.example` to `.env` and edit as needed
  - MongoDB required (local or Atlas)

## Project-Specific Patterns & Conventions

- **API endpoints**: Defined in backend/src/routes/\* (RESTful, e.g. `/api/alerts`, `/api/sentinels`)
- **WebSocket events**: Real-time updates via Socket.io (`new-alert`, `alert-verified`, etc.)
- **Data models**: See backend/README.md for Sentinel and Alert schemas
- **SMS integration**: Mock by default; enable real Huawei SMN by installing SDK and updating `.env`
- **Security**: Helmet.js, CORS, rate limiting, Mongoose validation
- **Frontend routing**: src/pages/_ and src/components/_
- **UI library**: shadcn-ui (see src/components/ui/\*)

## Integration Points

- **MongoDB**: Connection string in backend/.env
- **Huawei Cloud SMN**: Optional, configure via backend/.env
- **Socket.io**: Used for dashboard live updates
- **Raspberry Pi**: Sends alerts via HTTP POST to backend

## Troubleshooting & Monitoring

- **Backend logs**: Sentinel registrations, alerts, WebSocket connections, SMS notifications, errors
- **Common issues**: MongoDB connection, port conflicts (see backend/README.md for commands)

## Key Files & Directories

- Frontend: `src/`, `src/pages/`, `src/components/`, `src/services/`
- Backend: `backend/src/`, `backend/src/routes/`, `backend/src/controllers/`, `backend/src/models/`, `backend/src/services/`
- Data models: `backend/src/models/Alert.ts`, `backend/src/models/Sentinel.ts`
- WebSocket: `backend/src/server.ts`, `backend/src/services/huaweiSMN.ts`
- Raspberry Pi: `backend/examples/raspberry_pi_client.py`, `backend/raspberry-pi/sentinel_main.py`

---

For more details, see `README.md` and `backend/README.md`. Update this file if major architecture or workflow changes occur.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JoshNuku) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
