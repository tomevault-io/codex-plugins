# Copilot Instructions - Tick-Fix

## Project Overview
This is a computer vision project focused on facial landmark detection using MediaPipe's FaceMesh solution with OpenCV for real-time webcam processing. The project specifically tracks mouth-related facial landmarks to detect facial tics or movements.

## Architecture & Key Components

### Main Processing Pipeline (`main.py`)
1. **MediaPipe FaceMesh** - Handles facial landmark detection with 468+ 3D landmarks
2. **OpenCV** - Manages webcam capture (currently configured for camera index 1) and frame rendering
3. **Landmark Filtering** - Focuses on specific mouth landmarks defined in `tick_landmarks`: `mouth_left`, `mouth_right`, `mouth_upper`, `mouth_lower`

### Critical Landmark System
- **LANDMARK_NAMES**: Dictionary mapping MediaPipe landmark indices to human-readable names
- Key mouth landmarks: 61 (left), 291 (right), 13 (upper), 14 (lower)
- The system uses `refine_landmarks=True` to get detailed lip/iris tracking
- Only landmarks in `tick_landmarks` list are displayed with labels to reduce visual clutter

## Development Workflows

### Running the Application
```bash
python main.py
```
- Uses **camera index 1** (not default 0) - adjust `cv2.VideoCapture(1)` if needed
- Press **ESC** to exit the application
- Virtual environment already configured at `.venv/`

### Key Dependencies
- `mediapipe==0.10.14` - Face mesh detection
- `opencv-contrib-python==4.12.0.88` - Video capture and rendering
- `numpy==2.2.6` - Array operations for landmark coordinates
- Python 3.12.7 in virtual environment

## Project-Specific Conventions

### Landmark Coordinate System
- Landmarks are normalized (0.0 to 1.0) and must be multiplied by frame dimensions:
  ```python
  x = int(landmark.x * w)  # w = frame width
  y = int(landmark.y * h)  # h = frame height
  ```

### Display Patterns
- Black background rectangles behind text labels for visibility
- White text (255, 255, 255) on dark backgrounds
- Font: `cv2.FONT_HERSHEY_SIMPLEX` at size 0.3 for named landmarks
- Only mouth landmarks are labeled to keep interface clean

### MediaPipe Configuration
- `max_num_faces=1` - Single face tracking for performance
- `refine_landmarks=True` - Enhanced detail for lips/iris
- `min_detection_confidence=0.5` and `min_tracking_confidence=0.5` - Balanced sensitivity

## Code Structure Notes

### Frame Processing Order
1. Capture frame from webcam
2. Convert BGR → RGB (MediaPipe requirement)
3. Process with FaceMesh
4. Convert RGB → BGR (OpenCV display requirement)
5. Draw mesh tesselation and contours
6. Overlay landmark labels for filtered points
7. Display and check for ESC key

### Adding New Tracked Landmarks
To track additional facial features:
1. Find the landmark index from [MediaPipe's documentation](https://github.com/google/mediapipe/blob/master/mediapipe/modules/face_geometry/data/canonical_face_model_uv_visualization.png)
2. Add to `LANDMARK_NAMES` dictionary with descriptive name
3. Add the name string to `tick_landmarks` list to enable display

## Integration Points
- **Webcam**: Direct OpenCV access via `cv2.VideoCapture()`
- **MediaPipe**: Cloud-free, on-device processing
- **Display**: Real-time OpenCV window with ESC key exit

## Notes
- `main2.py` exists but is currently empty - placeholder for future experiments
- Virtual environment is Windows-specific (`.venv/Scripts/python.exe`)
- No test suite currently implemented
- No requirements.txt - dependencies managed via pip in venv

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Benjythebee)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Benjythebee)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
