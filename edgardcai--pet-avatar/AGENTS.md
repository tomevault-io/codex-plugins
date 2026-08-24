# Pet Avatar AI Copilot Instructions

## Project Overview

This is a **bilingual (Chinese UI/docs, English code) AI pet companion system** with:
- **Dual backend**: Node.js (primary) + FastAPI (alternative) sharing same API contracts
- **Frontend**: React 18 + Three.js for 3D pet visualization
- **AI Engine**: DeepSeek-V3 via SiliconFlow API for personality-driven conversations
- **Core Feature**: HEXACO personality system (6 dimensions) that drives pet behavior, memory retention, and emotional responses

## Architecture & Data Flow

### Service Boundaries
```
Frontend (React + Three.js)
  ↓ HTTP REST
Backend-Node (Express + Prisma)
  ↓ External API
SiliconFlow (DeepSeek-V3)
```

- **backend-node/** is the **production backend** (TypeScript + Prisma + SQLite/PostgreSQL)
- **backend-fastapi/** is an **experimental alternative** (Python + SQLAlchemy)
- Both expose: `/api/pets`, `/api/persona`, `/api/memories`, `/ws` (WebRTC signaling)
- WebSocket at `/ws` provides **WebRTC signaling only** (not real-time chat)

### Key Data Models (Prisma Schema)
- **Pet**: Core entity with `name`, `species` (cat/dog), `breed`, `appearance` (JSON color/size)
- **Persona**: 1:1 with Pet, stores HEXACO values (0.0-1.0): `honesty`, `emotionality`, `extraversion`, `agreeableness`, `conscientiousness`, `openness`
- **Memory**: 1:N with Pet, stores conversation snippets with `importance` score (0.0-1.0)

## Critical Workflows

### Development Setup
```bash
# Quick start all services
./quick-start.sh

# Manual backend setup
cd backend-node
npm install
npx prisma generate && npx prisma db push
npm run dev  # Runs tsx watch on port 3000

# Frontend setup
cd frontend
npm install
npm run dev  # Vite dev server on port 5173
```

### AI Service Integration
**Location**: [backend-node/src/services/aiService.ts](backend-node/src/services/aiService.ts), [backend-node/src/config/ai.ts](backend-node/src/config/ai.ts)

- **API Key**: Set `SILICONFLOW_API_KEY` in `.env` (defaults to hardcoded key in [config/ai.ts](backend-node/src/config/ai.ts#L3))
- **Model**: `deepseek-ai/DeepSeek-V3` via `https://api.siliconflow.cn/v1`
- **Context Management**: `PetConversationEngine` auto-loads pet data + memories from DB, constructs system prompts with:
  - HEXACO personality traits (affects response style)
  - Recent memories (importance > 0.6 prioritized)
  - Persona-specific dialogue examples
- **Response Format**: Enforces **15-30 character responses** in Chinese to mimic realistic pet communication

### Pet Replication System
**Location**: [backend-node/src/services/replicationService.ts](backend-node/src/services/replicationService.ts)

Two creation modes:
1. **Replicate Real Pet**: 5-question survey (`REPLICATION_QUESTIONNAIRE` in [config/ai.ts](backend-node/src/config/ai.ts)) → AI infers HEXACO scores + generates backstory
2. **Generate New Pet**: Natural language description → AI extracts personality + appearance

**Endpoint**: `POST /api/pets/replicate` (body: `{answers: string[], description?: string}`)

## Project-Specific Conventions

### HEXACO Personality Mapping
**Not standard Big Five** - uses 6 dimensions:
- `honesty`: 诚实守信 vs 偶尔调皮
- `emotionality`: 情感丰富 vs 相对理性
- `extraversion`: 喜欢社交 vs 比较内向
- `agreeableness`: 温顺友好 vs 有主见
- `conscientiousness`: 守规矩 vs 随性自由
- `openness`: 好奇探索 vs 喜欢熟悉

High values (>0.7) in `extraversion` → faster tail-wag animations in [ThreeScene.tsx](frontend/src/components/ThreeScene.tsx#L85-L100)

### 3D Model System
**Location**: [frontend/src/components/ThreeSceneGLTF.tsx](frontend/src/components/ThreeSceneGLTF.tsx)

- Models stored in `frontend/public/models/cat/` and `frontend/public/models/dog-gltf/`
- Supports both procedural geometry (deprecated [ThreeScene.tsx](frontend/src/components/ThreeScene.tsx)) and GLTF loaders
- **Color Mapping**: Persona traits influence model colors:
  - High `emotionality` → warmer hues
  - High `openness` → brighter colors
- **Animations**: Idle breathing + persona-driven ear/tail movements (see [3D_MODELS_SETUP.md](3D_MODELS_SETUP.md))

### Emotional Value System
**Location**: [backend-node/src/services/emotionalService.ts](backend-node/src/services/emotionalService.ts)

- Analyzes user input for 7 emotion types: `happy`, `sad`, `angry`, `anxious`, `lonely`, `neutral`, `excited`
- Identifies needs: `needsAffection`, `needsEncouragement`, `needsListening`
- Returns **3D behavior suggestions** (e.g., "wag tail slowly", "purr softly")
- **Endpoint**: `POST /api/pets/:id/emotion` (auto-saves high-importance memories)

## Common Pitfalls

1. **Database Migrations**: Always run `npx prisma generate && npx prisma db push` after schema changes in [backend-node/prisma/schema.prisma](backend-node/prisma/schema.prisma)

2. **CORS Issues**: Backend defaults to `allow_origins: ["*"]` - lock down in production

3. **AI API Rate Limits**: DeepSeek-V3 has token limits - use `AI_CONFIG.defaultParams.max_tokens` to control

4. **WebRTC Signaling**: The `/ws` endpoint is **minimal** (see [backend-node/src/index.ts](backend-node/src/index.ts#L22-L38)) - only echoes SDP/ICE, no TURN server support

5. **Persona Presets**: Don't modify `PERSONA_PRESETS` in [config/ai.ts](backend-node/src/config/ai.ts#L14-L90) without updating corresponding UI in [PetList.tsx](frontend/src/components/PetList.tsx)

## Key Files to Reference

- [backend-node/src/routes/pets.ts](backend-node/src/routes/pets.ts) - All pet CRUD + AI conversation endpoints
- [backend-node/src/services/aiService.ts](backend-node/src/services/aiService.ts) - Core conversation engine with context management
- [frontend/src/components/PetChat.tsx](frontend/src/components/PetChat.tsx) - Chat UI with quick-reply buttons
- [README.md](README.md) - Full feature list and Chinese documentation

## Testing & Debugging

- **Backend Logs**: Watch `console.error()` calls in services - errors propagate to HTTP 500
- **AI Responses**: Check `aiService.ts` `callAI()` method for raw API responses
- **3D Issues**: Enable Three.js DevTools extension, inspect `petRef.current` in console
- **DB Inspection**: Use `npx prisma studio` to browse SQLite data visually

## Environment Variables

```bash
# backend-node/.env
DATABASE_URL="file:./dev.db"  # SQLite (dev) or postgresql://... (prod)
SILICONFLOW_API_KEY="sk-..."  # Override hardcoded key
PORT=3000

# frontend/.env
VITE_SIGNALING_WS="ws://localhost:3000/ws"  # WebRTC signaling endpoint
```

---
> Source: [Edgardcai/pet-avatar](https://github.com/Edgardcai/pet-avatar) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-21 -->
