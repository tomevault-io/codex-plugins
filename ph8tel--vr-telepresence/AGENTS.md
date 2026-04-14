# VR Stereo Streaming System - AI Coding Instructions

## Architecture Overview

This is a **WebXR-based VR video streaming system** that streams real-time stereo camera feeds from an OAK-Pro camera (connected to Raspberry Pi 5) to VR headsets (Meta Quest Pro) over WebRTC.

**Three core components:**
1. **[stereo_camera.py](../stereo_camera.py)** - DepthAI OAK-Pro camera interface (CAM_B=left, CAM_C=right)
2. **[vr_server.py](../vr_server.py)** - WebRTC signaling server (aiohttp + aiortc)
3. **[index.html](../index.html)** - WebXR client (runs in Meta Quest Browser)

**Data flow:**
- **Video path**: OAK-Pro → Pi 5 → WebRTC (2 tracks) → Quest headset → WebGL stereo rendering
- **Control path**: Quest controllers/head tracking → WebRTC Data Channel → Pi 5 → optional TCP → Pi 4 servo controller

## Critical Hardware Context

- **Stereo baseline**: OAK-Pro has 75mm camera separation (baseline)
- **User IPD**: Target user has 71mm IPD → requires 4mm compensation
- **IPD adjustment**: Applied in WebGL shader via `uIPDOffset` uniform (~2.67% horizontal shift per eye)
- **Camera mapping**: `CAM_B` (left), `CAM_C` (right) on OAK-Pro board
- **Video format**: Rectified grayscale → converted to RGB for WebRTC

## Development Workflow

### Running the server
```bash
# On Raspberry Pi 5 with OAK-Pro connected
python3 vr_server.py
```
- Serves HTTP on port 8080 (expects Cloudflare Tunnel for HTTPS)
- Creates WebRTC offer with 2 video tracks + 1 data channel
- Optionally connects to Pi 4 servo controller at `192.168.1.138:9090` (non-blocking)

### Testing without VR headset
- Open `http://<pi-ip>:8080` in desktop browser
- Click "Test View (No VR)" to see side-by-side stereo streams
- Check debug log panel for WebRTC connection status

### Debugging WebRTC issues
1. Check browser console for: ICE state, connection state, track events
2. Server logs show: offer/answer SDP, track creation, data channel state
3. Common issues:
   - No video: Verify both tracks added via `addTransceiver()` in [vr_server.py](../vr_server.py#L120-L121)
   - Data channel not opening: Check SDP includes `m=application` line
   - Frame drops: Adjust `maxSize` in camera output queues ([stereo_camera.py](../stereo_camera.py#L21-L22))

## Code Conventions & Patterns

### WebRTC track handling
- **Server creates offer** (not client) because server sends video
- Use `addTransceiver(track, direction="sendonly")` for explicit control
- Never use `addTrack()` - it doesn't provide direction control

### Camera frame synchronization
- `StereoDepth.build()` in [stereo_camera.py](../stereo_camera.py#L16-L20) ensures left/right sync
- `get_frames()` pulls from both queues → inherently synchronized by hardware

### WebXR rendering loop
- Runs at headset refresh rate (72-120 Hz)
- Head pose sent every frame via data channel ([index.html](../index.html#L469-L495))
- Controller data sent every 3rd frame to reduce bandwidth ([index.html](../index.html#L430))
- WebGL textures updated from `<video>` elements using `texImage2D()`

### IPD correction math
```javascript
// In fragment shader (index.html line ~284)
const ipdAdjustment = 0.0267;  // 4mm / 75mm / 2
const offset = (view.eye === "left") ? ipdAdjustment : -ipdAdjustment;
```
- Shifts left eye texture right (+), right eye left (-) to match user IPD
- **Never modify baseline** - it's a physical camera property

### Data channel message format
All messages are JSON with `type` field:
```javascript
// Head tracking (sent every frame)
{ type: "pose", timestamp: ms, position: {x,y,z}, orientation: {x,y,z,w} }

// Controller inputs (sent every 3rd frame)
{ type: "controller", timestamp: ms, leftJoystick: {x,y}, rightJoystick: {x,y}, buttons: {...} }
```

### Error handling pattern
- Camera errors: Log to console, raise exception → server will restart track
- WebRTC errors: Log to page via `logToPage()` function for user visibility
- Servo connection: **Non-blocking** - server continues without servo control if Pi 4 unreachable

## External Dependencies

- **Python**: `depthai`, `aiortc`, `aiohttp`, `cv2`, `numpy`
- **JavaScript**: WebXR Device API, WebRTC, WebGL (no frameworks)
- **Network**: Requires HTTPS for WebXR (use Cloudflare Tunnel or localhost exception)
- **Hardware**: OAK-Pro camera, Meta Quest Pro headset

## Browser Compatibility

- **Primary target**: Meta Quest Browser (built-in on Quest Pro)
- **Known issues**: 
  - Wolvic browser: WebXR not available
  - Desktop browsers: Can test video streams but not VR mode
  - HTTP sites: WebXR requires HTTPS except on localhost

## Common Modifications

### Adjusting video quality
- Resolution: Change `size=(1280, 720)` in [vr_server.py](../vr_server.py#L71)
- Frame rate: Modify `await asyncio.sleep(1/30)` in [vr_server.py](../vr_server.py#L28)

### Adding new controller mappings
- Edit `sendControllerData()` in [index.html](../index.html#L497-L562)
- Quest Pro button indices: 0=trigger, 1=grip, 4=A/X, 5=B/Y

### Changing servo controller address
- Update `host` parameter in [vr_server.py](../vr_server.py#L159) `on_startup()`

## Why This Architecture?

- **Why rectification?**: Raw stereo would show misaligned images → eye strain in VR
- **Why server-creates-offer?**: Server is media source (sendonly) → must create offer with tracks
- **Why data channel in offer?**: Ensures negotiation before client tries to send pose data
- **Why TCP to Pi 4?**: Servo control needs reliability, not speed (vs UDP)
- **Why manual timestamps?**: aiortc requires monotonic PTS for video encoding

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ph8tel) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
