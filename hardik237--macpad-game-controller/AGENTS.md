# Copilot Instructions — MacPad Game Controller

## Project Overview

MacPad turns an iPhone into a wireless, low-latency game controller for macOS. Users connect their Mac and iPhone to the same WiFi, then open a URL on iPhone Safari to access a virtual gamepad UI that sends key presses to the Mac via WebSocket. The primary use case is playing retro games on [Afterplay.io](https://afterplay.io/).

## Architecture

```
iPhone (React SPA in Safari)
  ↕ WebSocket (ws://<MAC_IP>:8000/ws)
macOS Host
  ├─ Docker Container (port 8080)
  │   ├─ Nginx (serves static React build)
  │   └─ Vite-built React app
  └─ Native Python Daemon (port 8000)
      ├─ FastAPI + uvicorn (WebSocket server)
      └─ pynput (keyboard simulation via macOS Accessibility API)
```

- **Frontend** is served from Docker via Nginx. It is a React SPA that connects to the Python daemon over WebSocket.
- **Backend** runs natively on macOS (not in Docker) because it needs direct access to the macOS Accessibility API for keyboard simulation.
- **`start.sh`** orchestrates both: starts the Python venv + server, then Docker Compose for the frontend.

## Tech Stack

| Layer          | Stack                                                |
|----------------|------------------------------------------------------|
| Frontend       | React 18, Vite 4, plain CSS with custom properties   |
| Backend        | Python 3, FastAPI, uvicorn, pynput                   |
| Infrastructure | Docker (multi-stage: node → nginx:alpine), Docker Compose 3.8 |
| Communication  | WebSocket (native browser API, no socket.io)         |

## Code Conventions

### General
- Keep dependencies minimal — no Redux, no routing library, no HTTP client, no CSS-in-JS.
- Single-file component pattern: `App.jsx` contains the full UI.
- localStorage is the persistence layer for all client state (settings, layout, key mappings).

### Frontend (React / JavaScript)
- **Functions & variables**: camelCase (`handlePress`, `customKeys`, `editLayoutMode`).
- **CSS classes**: kebab-case (`dpad-btn`, `action-buttons`, `fullscreen-btn`).
- **CSS variables**: kebab-case with `--` prefix (`--bg-color`, `--highlight`, `--surface-color`).
- **Component files**: PascalCase (`App.jsx`); config files: lowercase (`vite.config.js`).
- Use the native `WebSocket` API — no wrapper libraries.
- Always handle both touch and mouse events for cross-device compatibility. Use `onTouchStart/End`, `onMouseDown/Up`, and `onMouseLeave` together.
- Debounce synthesized mouse events on iOS Safari (track `lastTouchTime`, ignore mouse events within ~500ms of touch).
- Use `navigator.vibrate` and Web Audio API for haptic/audio feedback.

### Backend (Python)
- **API actions**: lowercase strings (`"press"`, `"release"`, `"configure"`, `"ping"`, `"auth"`).
- Use a two-tier key mapping system: `KEY_MAPS` for defaults, `USER_MAPPINGS` for runtime overrides per player.
- Wrap blocking `pynput` calls in `run_in_threadpool` to avoid blocking the async event loop.
- WebSocket message format: `{"action": "<action>", "key": "<key>"}` (player is determined by auth, not sent by client).
- **Player authentication**: random 4-digit codes generated per server session. Codes persist across uvicorn reloads via environment variables. Clients must send `{"action": "auth", "code": "<4-digit>"}` before any other action.
- CORS is open (`allow_origins=["*"]`) — this is intentional because the app runs on a private LAN.
- Event-level logs (press/release) are `debug`; connection and auth logs are `info`.

### CSS / UI Design
- Glass-morphism style using `backdrop-filter: blur()`, rgba surfaces, and gradient text.
- Color system defined via CSS custom properties in `:root`.
- Global touch prevention: `user-select: none`, `touch-action: none`, `-webkit-touch-callout: none`.
- Depth effect using combined inset + outer `box-shadow`.

## Project Structure

```
frontend/           # React SPA (Vite)
  src/
    App.jsx         # Main component — all UI and logic
    main.jsx        # React entry point
    index.css       # Global styles + CSS variables
  index.html        # HTML shell
  vite.config.js    # Vite config
  package.json

host_daemon/        # Python WebSocket server (runs natively on macOS)
  server.py         # FastAPI app with WebSocket handler + pynput keyboard control
  requirements.txt  # Python dependencies

nginx/
  nginx.conf        # SPA routing config (try_files fallback to index.html)

Dockerfile          # Multi-stage build: node:18-alpine → nginx:alpine
docker-compose.yml  # Frontend service definition
start.sh            # One-click orchestration script
```

## Running the Project

### Production (one command)
```bash
./start.sh
# Starts Python daemon + Docker frontend
# Prints iPhone URL: http://<MAC_IP>:8080
```

### Frontend Development
```bash
cd frontend && npm install && npm run dev
```

### Backend Development
```bash
cd host_daemon
python3 -m venv venv && source venv/bin/activate
pip install -r requirements.txt
python3 server.py   # uvicorn with reload=True
```

### Prerequisites
- macOS with Accessibility permissions granted to Terminal.
- Docker Desktop running.
- Mac and iPhone on the same WiFi network.

## Key Design Decisions
- **WebSocket over HTTP**: real-time bidirectional comms for minimal latency.
- **Native Python daemon (not Dockerized)**: requires direct macOS Accessibility API access.
- **Docker for frontend only**: isolated, reproducible Nginx serving.
- **No state management library**: app is simple enough for React `useState` + localStorage.
- **Player code authentication**: random 4-digit codes per session lock each phone to P1 or P2, preventing players from switching controls. Codes are cached in `localStorage` for auto-reconnect; cleared on logout.
- **Multi-player support**: separate key mappings for Player 1 (arrow keys) and Player 2 (WASD).
- **Drag-and-drop layout editor**: accommodates different iPhone screen sizes and orientations.
- **Cross-platform compatibility**: touch + mouse event handling for iOS/Android and desktop browsers.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hardik237) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
