<!-- SentinelAI School Security Monitoring System -->

## Project Overview
Real-time AI-powered surveillance system using YOLOv8 for zone-based threat detection in schools.

## Tech Stack
- **Backend**: FastAPI (Python 3.10+)
- **AI Worker**: YOLOv8 (ultralytics)
- **Frontend**: Next.js + Tailwind CSS
- **Real-time**: WebSockets

## Zone Types
- `outgate` - Vehicle detection, gate accidents
- `corridor` - Crowd formation, fights
- `school_ground` - Crowd monitoring, violence
- `classroom` - Mobile phone usage

## Key Commands
```bash
# Backend
cd backend && uvicorn main:app --reload --port 8000

# Dashboard
cd dashboard && npm run dev

# Test detection
cd ai_worker && python test_worker.py <zone> <video_path>
```

## API Endpoints
- `POST /event` - Submit detection event
- `GET /api/zones` - Zone definitions
- `GET /api/cameras` - Camera configs
- `GET /video/{id}` - MJPEG stream
- `WS /ws/alerts` - Real-time alerts

---
> Source: [exodus2512/CCTvAnalysis](https://github.com/exodus2512/CCTvAnalysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-29 -->
