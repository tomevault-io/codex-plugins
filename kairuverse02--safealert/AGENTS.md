# SAFEAlert AI Coding Agent Instructions

## Project Overview
SAFEAlert is a Next.js 15 (App Router) real-time monitoring application that establishes **WebRTC peer-to-peer connections** between a **guardian** (monitor) and a **dependent** (patient) using **Supabase** for signaling persistence. The guardian receives live video/audio streams and performs computer vision-based motion detection, sound analysis, and perimeter breach detection.

**Core Architecture**: Guardian-Dependent P2P model with polling-based signaling (no realtime subscriptions for signal exchange).

## Critical Architecture Knowledge

### 1. WebRTC Signaling Flow (Non-Standard Pattern)
- **Signaling is polling-based, NOT WebSocket/Realtime**
- Supabase's `pairing_rooms` table stores offer/answer signals as JSONB
- Guardian creates room → Dependent scans QR code → establishes P2P connection
- **Key quirk**: Answer signals with candidates-only updates are **merged** with existing SDP to prevent overwriting (see [route.ts](src/app/api/signaling/[id]/route.ts#L38-L96))
- Dependent initially sends answer with `audio: false` to avoid m-line reordering errors ([DEBUGGING_ANALYSIS.md](DEBUGGING_ANALYSIS.md))

**Files**: [WebRTCContext.tsx](src/contexts/WebRTCContext.tsx), [api/signaling/[id]/route.ts](src/app/api/signaling/[id]/route.ts), [db/create_pairing_rooms.sql](db/create_pairing_rooms.sql)

### 2. Dual-Role Component Structure
- `/admin/*` routes: **Guardian** (monitoring dashboard, receives video/audio stream)
- `/client/*` routes: **Dependent** (sends video/audio, responds to commands)
- Auth middleware enforces authenticated access ([middleware.ts](middleware.ts), [lib/supabase/middleware.ts](src/lib/supabase/middleware.ts))

### 3. Monitoring Modes (Guardian-Side)
Defined in [types/index.ts](src/types/index.ts):
- `idle`: No monitoring active
- `perimeter_setup`: Guardian draws virtual boundary on video canvas
- `perimeter_monitoring`: Detects motion outside patient boundary
- `patient_monitoring`: Detects patient motion/stillness inside boundary

**Motion detection algorithm**: Frame-by-frame grayscale diff with configurable thresholds ([lib/motion.ts](src/lib/motion.ts), [lib/constants.ts](src/lib/constants.ts))

### 4. Guardian-Dependent Communication Patterns
- **Guardian → Dependent**: Commands via `guardian_command` field (e.g., `start_monitor`, `stop_monitor`)
- **Dependent → Guardian**: Events via `guardian_event` JSONB (e.g., `patient_sound`, `patient_mic_test_start`, `sos_triggered`)
- **Dependent actions**: Simple string signals like `water`, `bathroom`, `sos_start` via `dependent_action` field

**Polling frequency**: Dependent polls guardian commands every 2 seconds ([PatientScanner.tsx](src/app/client/components/PatientScanner.tsx))

### 5. Known WebRTC Timing Issues
From [DEBUGGING_ANALYSIS.md](DEBUGGING_ANALYSIS.md) and [FIXES_APPLIED.md](FIXES_APPLIED.md):
- **8-second redirect timeout**: Dependent redirects to dashboard if connection not stable within 8s (can cause premature disconnects)
- **Fix pattern**: Check for active tracks/stable connection state before redirect
- **Video readyState stuck at HAVE_NOTHING**: Indicates connection died after tracks were received
- **Connection state monitoring**: Always log `connectionState` transitions for debugging

## Developer Workflow

### Environment Setup
1. Copy `.env.example` → `.env.local`
2. Fill in 3 Supabase keys: `NEXT_PUBLIC_SUPABASE_URL`, `NEXT_PUBLIC_SUPABASE_ANON_KEY`, `SUPABASE_SERVICE_ROLE_KEY`
3. Set `NEXT_PUBLIC_SITE_URL` (e.g., `http://localhost:3000` for local dev)
4. Run SQL script: [db/create_pairing_rooms.sql](db/create_pairing_rooms.sql) in Supabase SQL Editor

### Commands
```bash
npm install          # Install dependencies
npm run dev          # Start dev server (port 3000)
npm run build        # Production build (check for type errors)
npm run lint         # ESLint check (must pass with 0 errors for deployment)
```

### Testing WebRTC Locally
1. Start dev server: `npm run dev`
2. Open two browser windows (or one incognito + one normal):
   - Window 1: Login as guardian → `/admin/dashboard` → Create pairing room (QR code)
   - Window 2: Login as dependent → `/client` → Scan QR code → Establish connection
3. Guardian sends `start_monitor` command → Dependent starts camera/mic

## Project-Specific Conventions

### Type Safety
- **Strict TypeScript**: No `any` types allowed (enforced by ESLint)
- Union types for specific sets: `LogEntryType = "sound" | "patient_motion" | "bathroom" | ...`
- Type guards for external data: Check `typeof` before accessing properties on JSONB responses

### Supabase Client Patterns
- **Server Components/Actions**: Use `await createClient()` from [lib/supabase/server.ts](src/lib/supabase/server.ts)
- **Client Components**: Use `createClient()` from [lib/supabase/client.ts](src/lib/supabase/client.ts)
- **API Routes (privileged)**: Use `createServiceClient()` from [lib/supabase/service.ts](src/lib/supabase/service.ts) for bypassing RLS

### Motion Detection Tuning
Constants in [lib/constants.ts](src/lib/constants.ts):
- `PATIENT_MOTION_THRESHOLD`: Pixel brightness diff (default: 6)
- `PATIENT_MIN_MOTION_PIXELS`: Min changed pixels (default: 4)
- `PERIMETER_MOTION_THRESHOLD`: For boundary breach detection (default: 30)
- **Calibration**: Guardian can dynamically adjust thresholds at runtime ([MonitoringSystem.tsx](src/app/admin/components/MonitoringSystem.tsx))

### Audio/Sound Detection
- Uses Web Audio API with `AnalyserNode` ([hooks/useSoundDetection.ts](src/hooks/useSoundDetection.ts))
- **Threshold**: `-30 dB` for sustained sound, `-12 dB` for cough/spike detection
- **Cooldown**: 3s between alerts to avoid spam
- Must call `initAudio()` after user gesture (browser autoplay policy)

### Component Patterns
- **Custom Hooks**: Encapsulate device access (`useCamera`, `useAudio`, `useSoundDetection`)
- **Refs for WebRTC**: Store `Peer` instances, `MediaStream`, polling intervals in refs (not state)
- **Canvas Drawing**: Always check `videoRef.current.readyState >= HTMLMediaElement.HAVE_CURRENT_DATA` before drawing ([MonitoringSystem.tsx](src/app/admin/components/MonitoringSystem.tsx#L378))

## Common Pitfalls

1. **Unescaped HTML entities in JSX**: Use `&quot;`, `&apos;` instead of quotes/apostrophes in attribute values
2. **Async middleware**: `updateSession()` must be awaited in [middleware.ts](middleware.ts)
3. **ICE candidate merging**: Never overwrite existing `answer_signal.sdp` when adding candidates ([route.ts](src/app/api/signaling/[id]/route.ts))
4. **Peer cleanup**: Always destroy peer and stop tracks in cleanup/unmount ([WebRTCContext.tsx](src/contexts/WebRTCContext.tsx#L64-L103))
5. **Email redirect URLs**: Use `new URL(origin).origin` to strip path segments, avoiding `/role/auth/confirm` 404s ([lib/auth-actions.ts](src/lib/auth-actions.ts#L35-L42))

## Debugging Commands
```bash
# Check TypeScript errors
npx tsc --noEmit

# Check lint (must be 0 errors)
npm run lint

# View Supabase logs (if using CLI)
supabase logs

# Test scripts (check guardian events)
node scripts/check_guardian_event.js
node scripts/test_confirm.js
```

## Key Files Reference
- **WebRTC Core**: [WebRTCContext.tsx](src/contexts/WebRTCContext.tsx) (653 lines)
- **Guardian Monitor**: [MonitoringSystem.tsx](src/app/admin/components/MonitoringSystem.tsx) (1070 lines)
- **Dependent Scanner**: [PatientScanner.tsx](src/app/client/components/PatientScanner.tsx) (449 lines)
- **Signaling API**: [api/signaling/[id]/route.ts](src/app/api/signaling/[id]/route.ts)
- **Auth Actions**: [lib/auth-actions.ts](src/lib/auth-actions.ts)
- **Motion Detection**: [lib/motion.ts](src/lib/motion.ts), [lib/constants.ts](src/lib/constants.ts)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kairuverse02) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
