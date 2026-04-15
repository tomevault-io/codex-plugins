# Shorts — AI Video Generation & Autonomous Publishing System

AI 기반 영상 생성 → 에이전트 배포 → 멀티 플랫폼 퍼블리싱 → ROI 피드백 루프를 자동화하는 시스템.

## Architecture

```
┌─────────────┐   ┌──────────────┐   ┌──────────────────┐
│  Frontend    │   │   Backend    │   │  Agent Fleet     │
│  React/Vite  │──▶│  FastAPI     │◀──│  APK · EXE       │
│  :3000       │   │  :8000       │   │  heartbeat 30s   │
└─────────────┘   └──────┬───────┘   └──────────────────┘
                         │
              ┌──────────┼──────────┐
              │          │          │
         MySQL 8.4   Memurai    cloudflared
         :3306       :6379      shorts.songstock.art
```

**Pipeline**: Planning → GPU Creation → Agent Dispatch → Publishing → ROI Feedback

## Key Directories

| Directory | Purpose |
|-----------|---------|
| `backend/app/` | FastAPI 서버 — routers/, models/, schemas/, services/ |
| `frontend/src/` | React 관제 UI — components/, analytics/ |
| `agent/ShortsAgent/` | React Native 모바일 에이전트 |
| `agent/exe/` | Windows EXE 에이전트 |
| `vibe_code/` | 영상 템플릿 (Vibe Code 마크다운) |
| `scripts/` | PowerShell 자동화 (startup, build, smoke test) |
| `backend/db/migrations/` | Alembic SQL 마이그레이션 파일 |
| `docs/` | API 스펙, ERD, 테스트 시나리오, 운영 보고서 |

## Build & Run

```powershell
# 개발 환경 셋업 (Python 3.11, Node.js, MySQL, FFmpeg)
.\setup-dev-env.ps1

# 전체 시스템 시작 (MySQL + Backend + Frontend + cloudflared)
.\startshort.bat

# 백엔드 단독
cd backend
uvicorn app.main:app --host 0.0.0.0 --port 8000 --reload

# 프론트엔드 단독
cd frontend
npm run dev
```

## Test

```powershell
# 백엔드 유닛 테스트 (SQLite in-memory)
cd backend
pytest

# E2E smoke test
.\scripts\e2e_smoke.ps1

# Full auto long-run
.\scripts\c10_full_auto_longrun_smoke.ps1
```

## Database

- **ORM**: SQLAlchemy 2.x Mapped types + Pydantic schemas
- **Migration**: `cd backend && alembic revision --autogenerate -m "msg"` → `alembic upgrade head`
- **Pool**: size=20, max_overflow=30, recycle=1800s, pool_pre_ping=True
- **Soft deletes**: `status` 필드 (ACTIVE/INACTIVE/QUARANTINED)
- **Audit**: `write_audit_log()` 호출 필수 (보안 관련 변경 시)

## API Conventions

- Base path: `/api/v1`
- Auth: Bearer JWT (`Authorization` 헤더)
- Router → Service → Repository 패턴
- AI 프로바이더: Registry 패턴 (`services/providers/`)
- DB 세션: `Depends(get_db)` 의존성 주입

## Critical Constraints

| 항목 | 규칙 |
|------|------|
| 전화번호 | `010-1234-5678` 형식 정규화 필수 (10-11자리) |
| Agent 타입 | `"ask.exe"` → `"exe"`, 기본값 `"apk"` |
| SIM 한도 | 사용자당 최대 100개 (LRU 교체) |
| Wan2.1 프레임 | 반드시 4n+1 (17, 33, 49, 65, 81) |
| GPU 인코딩 | h264_nvenc 권장 (RTX 5060 Ti 16GB) |
| Risk scoring | 섀도우밴 감지 시 자동 격리 → 수동 복구 |

## Environment

`.env` 파일은 프로젝트 루트에 위치. 참조: `backend/app/core/config.py` (Pydantic Settings).

필수 변수: `DATABASE_URL`, `SECRET_KEY`, `ALGORITHM`
프론트엔드: `frontend/.env.example` 참조 (`VITE_API_HOST`, `VITE_API_BASE`)

## Specialized Agents

5개의 전문 에이전트가 `.github/agents/`에 정의되어 있습니다:

| Agent | 역할 | 주요 도메인 |
|-------|------|------------|
| `server-ops` | DB · 스케줄러 · 모니터링 · ROI | 백엔드 운영 전반 |
| `agent-fleet` | 디바이스 등록 · 하트비트 · 태스크 디스패치 | 에이전트 관리 |
| `publishing` | 멀티 플랫폼 업로드 · 컴플라이언스 | 배포 |
| `video-pipeline` | GPU 렌더링 · Wan2.1 · 영상 검증 | 영상 생성 |
| `vibe-code` | VibeCode 템플릿 · 메타데이터 | 컨셉 기획 |

## Documentation

| 문서 | 내용 |
|------|------|
| [docs/api-spec-v1.md](docs/api-spec-v1.md) | REST API 전체 스펙 |
| [docs/erd-v1.md](docs/erd-v1.md) | DB 엔티티 관계도 |
| [docs/apk-agent-api-spec.md](docs/apk-agent-api-spec.md) | Agent-Server 통신 프로토콜 |
| [docs/auth-policy-v1.md](docs/auth-policy-v1.md) | 인증·인가 정책 |
| [docs/integration-test-scenarios.md](docs/integration-test-scenarios.md) | 통합 테스트 시나리오 |
| [docs/c09-runbook-r09-report.md](docs/c09-runbook-r09-report.md) | 운영 런북 |

## Tech Stack

**Backend**: Python 3.11, FastAPI 0.116.1, SQLAlchemy 2.0, Alembic, PyJWT, APScheduler, httpx
**Frontend**: React 18.3, Vite 5.4, Recharts
**ML/Video**: PyTorch 2.6.0+cu124, diffusers 0.37, MoviePy, FFmpeg, OpenCV
**Audio**: edge-tts, librosa
**Infra**: MySQL 8.4 (Windows Service), Memurai (Redis-compatible), cloudflared tunnel

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/naverstop) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
