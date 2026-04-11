# Copilot Instructions: Hand Gesture Control System

## Project Overview
Computer vision-based hand gesture controller for keyboard/mouse inputs using MediaPipe + OpenCV. Users control games and applications via webcam-detected hand gestures.

## Architecture & File Organization

### Active Controllers
- **`keyboard.py`** - Unified keyboard controller (WASD + Arrow keys simultaneously)
- **`mouse.py`** - Dual-hand mouse controller with smoothed cursor movement and optimized tracking

## Key Technical Patterns

### State-Based Key Management
Controllers use `current_keys` set tracking with cooldown timers to prevent key spam:
```python
# CORRECT pattern (app.py, xkeys.py)
def update_keys(new_keys):
    global current_keys, last_key_time
    if current_time - last_key_time > KEY_COOLDOWN:
        # Release old, press new
        for key in current_keys - new_keys:
            keyboard.release(key)
        for key in new_keys - current_keys:
            keyboard.press(key)
```
**Never** use direct `keyboard.press()` without state tracking—causes stuck keys.

### Gesture Detection Logic
Finger state detection uses landmark Y-coordinates (lower Y = higher on screen):
```python
fingers = {
    'index': index_tip.y < landmarks[INDEX_FINGER_PIP].y,  # Tip above PIP = extended
    'middle': middle_tip.y < landmarks[MIDDLE_FINGER_PIP].y,
    # ... etc
}
```

Hand tilt for steering: Compare `INDEX_FINGER_TIP.x` to `WRIST.x` with deadzone threshold (0.03).

### Critical Constants
- `KEY_COOLDOWN = 0.1` - Key update throttle (seconds)
- `STEER_DEADZONE = 0.03` - Minimum hand tilt for steering
- `HAND_CONFIDENCE = 0.7` - MediaPipe detection threshold
- `max_num_hands=1` for gaming (keyboard), `max_num_hands=2` for mouse control

## Gesture Mappings

### Gaming Controls (keyboard.py)
- **Index finger only up** → Forward (W/↑)
- **Closed fist** → Backward (S/↓)
- **Hand tilt left** → Left (A/←)
- **Hand tilt right** → Right (D/→)
- **All fingers up** → Stop (releases all keys)
- Simultaneously presses both WASD and Arrow keys for maximum compatibility

### Mouse Controls (mouse.py)
- **Right hand**: Index pointing = move cursor (smoothed); Index+middle together = right-click
- **Left hand**: Index pointing = left-click; Index+middle together = double-click
- Optimized with 60 FPS capture, reduced buffer latency, and cursor smoothing
- Improved finger detection using DIP joints instead of MCP for better accuracy

## Common Modifications

### Adding New Gestures
1. Detect finger state in `fingers` dict
2. Add condition checking specific combination
3. Map to action in `active_keys` or `active_actions` set
4. Update status text for visual feedback

### Changing Key Bindings
Modify `CONTROLS` dict in `keyboard.py` for multi-key support:
```python
CONTROLS = {
    'forward': ['w', 'up', 'space'],  # Add space bar boost
    'backward': ['s', 'down'],
    'left': ['a', 'left'],
    'right': ['d', 'right']
}
```

### Adjusting Sensitivity
- Reduce `STEER_DEADZONE` for more responsive steering
- Increase `KEY_COOLDOWN` to slow input rate
- Lower `HAND_CONFIDENCE` if detection drops frequently

## Error Handling Pattern
Always wrap main loop in try-except to release keys on crash:
```python
try:
    # gesture processing
except Exception as e:
    print(f"Processing error: {str(e)}")
    update_keys(set())  # Release all keys
```

## Dependencies
Install via: `pip install opencv-python mediapipe pyautogui numpy keyboard`

Note: `keyboard` library requires admin/sudo on some systems for key simulation.

## Testing Workflow
1. Run controller: `python keyboard.py` (or `python mouse.py`)
2. Position hand in webcam view
3. Press 'q' in OpenCV window to exit safely
4. Verify all keys released after exit (test in notepad)

## Performance Notes
- **keyboard.py**: Optimized with 30 FPS, buffer size 1 for minimal latency
- **mouse.py**: Enhanced with 60 FPS, cursor smoothing (factor 0.5), and improved gesture detection
- Both use `model_complexity=0` for faster processing on lower-end hardware

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/JeetInTech)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/JeetInTech)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
