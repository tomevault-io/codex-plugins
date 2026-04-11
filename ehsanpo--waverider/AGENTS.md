# TunnelBeats Copilot Instructions

## Project Overview

**TunnelBeats** is a music-driven React Native racing game where players navigate procedurally-generated tunnels synchronized to audio. The core innovation is real-time tunnel geometry generation from audio frequency analysis, creating a gameplay experience directly tied to music structure.

### Key Architecture Pattern: Audio → Analysis → Geometry → Rendering

1. **Audio File** (MP3/WAV in `assets/audio/`) 
2. **Audio Analysis** (`lib/audioAnalyzer.ts`) extracts amplitude, frequency bands, BPM, and beat detection
3. **Tunnel Generation** (`lib/tunnelGenerator.ts`) creates segment geometry based on audio features
4. **Game Rendering** (`components/TunnelRenderer.tsx`) draws the tunnel, while gameplay (`app/(tabs)/game.tsx`) simulates physics and scoring

## Critical Workflows

### Adding Music Tracks (Most Common Task)
```bash
# 1. Place MP3 in assets/audio/Artist - Song Name.mp3
# 2. Run automatic import (analyzes and generates all 3 difficulties):
npm run import-audio

# For manual control or one-off testing:
npm run add-demo-track  # Creates a mock track with no actual audio
```

**Important**: `importAudioTracks.ts` expects filename pattern `Artist - Song Name.mp3` and auto-parses it. See `lib/audioAnalyzer.extractMetadataFromFilename()`.

### Dev Workflow
```bash
npm run dev              # Start Expo dev server
npm run typecheck       # TypeScript validation
npm run lint            # Code linting
```

### Regenerating Geometry
If tunnel generation algorithm changes:
```bash
npm run regenerate-levels  # Recalculates all level geometries
```

## Data Flow & Storage

### Supabase Database Schema
- **levels**: Stores track metadata + pre-generated `geometry_easy/normal/hard` (JSON)
- **scores**: User scores with `level_id`, `difficulty`, `score`, `distance`
- **users**: Anonymous or auth-linked player profiles

**Key pattern**: Geometry is **pre-computed and stored**, not generated at runtime. `lib/levels.ts` loads from DB.

### Game State Management
`GameState` (in `types/game.ts`) tracks: `lives`, `score`, `distance`, `combo`, `speed`, and `enemies`. State updates happen in the game loop (`game.tsx` lines ~150-200) at 60 FPS.

## Audio Analysis Deep Dive

`lib/audioAnalyzer.ts` uses Web Audio API:
- **FFT** on small segments → extracts bass/mid/treble bands
- **Beat Detection** via amplitude peak detection → timestamps used to narrow tunnel
- **BPM Estimation** from autocorrelation of energy curve

The analysis produces `AudioAnalysisResult` with arrays of 2000 data points per audio file.

## Tunnel Generation Logic

`lib/tunnelGenerator.ts`:
- Generates **60 segments per second** of gameplay
- Tunnel width varies based on:
  - **Bass waves** (slow oscillation) for major left/right movement
  - **Treble waves** (fast oscillation) for fine adjustments  
  - **Beat timestamps** narrow the tunnel near detected beats (difficulty-dependent)
  - **Amplitude** adds width pulsing
  
Each difficulty has hardcoded `DifficultyConfig` (easy: 240-300px width; hard: 140-200px).

**Pattern**: Difficulty doesn't change audio analysis, only tunnel parameters. Higher difficulty = narrower passages and faster speed changes.

## Game Mechanics

### Collision Detection
`game.tsx` (line ~450):
```typescript
// Ship bounding box vs tunnel segment bounding box
const shipLeft = shipX - SHIP_SIZE/2;
const shipRight = shipX + SHIP_SIZE/2;
if (shipLeft < tunnelSegment.leftX || shipRight > tunnelSegment.rightX) {
  // collision!
}
```

### Combo System
- Charging: +25% per enemy dodged (see `COMBO_CHARGE_PER_ENEMY`)
- Active: 1.5x speed multiplier for 8 seconds
- Combo affects music pitch (`Audio.Sound.setRate()`)

### Game Loop
Runs at 60 FPS in `requestAnimationFrame`. Each frame:
1. Update scroll offset (based on speed)
2. Check collisions
3. Spawn enemies periodically
4. Update combo/score
5. Render tunnel + ship + HUD

## Important Patterns & Constraints

### Type System
Use `Difficulty = 'easy' | 'normal' | 'hard'` (string literals, not enums) for DB compatibility.

### SVG Rendering with react-native-svg
`TunnelRenderer` builds SVG path strings dynamically. Path data is large; performance tested up to ~800 segments visible.

### Accelerometer Input
`setupAccelerometer()` in game.tsx reads device tilt (iOS/Android). Fallback: web uses keyboard arrow keys.

### Audio Playback
- Uses `expo-av` for playback control
- Supports rate changes (for combo multiplier effects)
- Paused state maintained in game state

### Asset Paths
- Audio files: `assets/audio/` (relative paths used in `audio_path` field)
- No Supabase Storage integration yet; files are local
- Migration script exists (`migrate-audio-to-storage.js`) if remote storage needed

## Common Modifications

| Task | Files | Notes |
|------|-------|-------|
| Change game speed | `game.tsx` (BASE_SPEED constant) | Also affects audio drift |
| Adjust difficulty balance | `lib/tunnelGenerator.ts` (DifficultyConfig) | Regenerate levels after change |
| Add new game feature | `types/game.ts` (GameState), `game.tsx` | Update game loop logic |
| Modify collision feel | `game.tsx` (COLLISION_TOLERANCE) | Haptic feedback also triggered here |
| New audio track | `assets/audio/` + `npm run import-audio` | Auto-parses filename for metadata |

## Testing Notes

- **Web browser**: Limited (no accelerometer). Good for UI/flow testing.
- **Emulator/Physical device**: Full game with tilt controls.
- Debug mode: Triple-tap game screen to toggle collision visualization.

## Key Dependencies

- **expo-av**: Audio playback with rate control
- **expo-sensors**: Accelerometer input
- **expo-router**: Navigation (stack-based)
- **@supabase/supabase-js**: DB + auth
- **react-native-svg**: SVG rendering for tunnel

## Environment Variables

Required in `.env` (Expo auto-loads as `EXPO_PUBLIC_`):
- `EXPO_PUBLIC_SUPABASE_URL`
- `EXPO_PUBLIC_SUPABASE_ANON_KEY`

## Gotchas & Pitfalls

1. **Geometry re-computation is slow** (30s+ for long tracks). Don't call in-app; use scripts.
2. **Beat detection can fail on silent audio** (demo track). Fallback uses generic beat pattern.
3. **SVG path performance degrades** with >1000 visible segments. Optimize segment culling if adding more.
4. **Combo state mutation**: Use spread operator when updating `comboState` in GameState.
5. **Audio sync drift**: If game speed changes, audio can desync. Always test with real audio.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ehsanpo)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/ehsanpo)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
