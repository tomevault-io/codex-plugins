# Squat Meter - AI Agent Instructions

## Architecture Overview

This is a **single-file PWA** with all logic in `index.html`. No build process, no bundler, no separate JS modules. All dependencies (TensorFlow.js, PoseNet, Chart.js) load from CDN.

**Core Flow:** Webcam → PoseNet → Angle Calculation → State Machine → Chart Updates → localStorage

## Critical Configuration (`CONF` object)

Located at the top of the `<script>` tag in `index.html`:

- `minConfidence: 0.2` - **Critical**: Lowered from 0.5 to fix detection issues. Don't increase without testing.
- `standThresh: 165°`, `squatThresh: 140°`, `deepThresh: 100°` - State machine thresholds for squat detection.
- `videoWidth/videoHeight` - Dynamically updated from actual webcam resolution.

## Key Patterns & Conventions

### State Management
Single `state` object holds all runtime state. Never split this across variables.
- `state.posenetModel` - Cached model instance (preloaded in `setup()` to avoid reload delay)
- `state.isDetecting` - Controls the main loop
- `state.currentSquatData` - Buffer for live squat being recorded

### Main Detection Loop
**Critical Fix**: Always check `video.readyState >= 3` before calling `net.estimateSinglePose()`. This prevents the `(0,0)` keypoint bug from incomplete video frames.

```javascript
if (video.readyState < 3) {
    state.animationFrameId = requestAnimationFrame(poseDetectionFrame);
    return;
}
```

### Squat Detection State Machine
`processPose()` implements a simple two-state machine:
1. **Standing** → **Squatting**: When angle < `squatThresh` (140°)
2. **Squatting** → **Standing**: When angle > `standThresh` (165°)

Uses whichever leg (right/left) has higher confidence score above `minConfidence`.

### Data Persistence
- `localStorage` key: `'squatHistory'` stores last 3 squats as JSON array
- Each entry: `{timestamp: ISO string, data: [{x: time, y: angle}, ...]}`
- Always maintain max 3 entries using `unshift()` + `pop()` pattern

## Development Workflow

**Local Server Required**: Webcam access needs HTTPS or localhost. Use:
```bash
python -m http.server
```

**No Git Operations**: User handles all git commands manually. Never `git add`, `git commit`, or `git push`.

### Debugging Patterns

Use the existing throttled logging pattern to avoid console spam:

```javascript
const now = Date.now();
if (now - state.lastLogTime > 2000) {  // Log every 2 seconds
    console.log("Debug info:", /* your data */);
    state.lastLogTime = now;
}
```

Key debugging points:
- `video.readyState` and dimensions - verify video is ready
- `pose.keypoints` - check confidence scores and positions
- Angle calculations - verify math before state changes

### Testing Approach

**No automated tests** - use manual testing workflow:

1. **Start/Stop Cycle**: Verify camera releases properly (check indicator light)
2. **Squat Detection**: Test all states - standing, squatting, transition thresholds
3. **Edge Cases**: 
   - Low light (tests `minConfidence` threshold)
   - Side angles (tests left/right leg selection)
   - Fast movements (tests state machine debouncing)
4. **Data Persistence**: Clear localStorage and verify history charts rebuild correctly
5. **PWA Install**: Test on mobile device for true PWA behavior

**Browser DevTools**: Use Application tab → localStorage to inspect `squatHistory` data structure

## Version Management

When making deployable changes, increment version in:
1. `GEMINI.md` - "Current Version" line
2. `index.html` - `.version` div (line ~107)
3. `index.html` - `<h1>` title (line ~108)

Current version: **1.0.1**

## Service Worker Strategy

`sw.js` is intentionally minimal (pass-through). Previous caching attempts caused stale code issues. Provides PWA installability without aggressive caching.

## Common Gotchas

1. **Video not mirrored in code**: `flipHorizontal: false` in `estimateSinglePose()` because CSS `scaleX(-1)` handles mirroring
2. **Chart animation disabled**: `animation: false` prevents performance issues during live updates
3. **Chart y-axis range**: 0-170° (not 0-180°) - adjusted for typical squat angles
4. **Angle calculation**: Uses law of cosines on Hip-Knee-Ankle triangle (always knee as vertex B)
5. **Keypoint selection**: Picks right or left leg based on higher average confidence score
6. **Chart update mode**: Use `chart.update('none')` for live chart to skip animations entirely

## Future Enhancement Areas

- Switch to MoveNet for better accuracy (PoseNet is dated)
- Extract JS to separate modules for maintainability
- Implement proper service worker caching strategy (stale-while-revalidate)
- Add settings panel for threshold configuration

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MartinNielsen)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/MartinNielsen)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
