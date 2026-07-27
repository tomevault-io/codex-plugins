# DroneAid 2026 - GitHub Copilot Instructions

## Project Overview

DroneAid 2026 is a disaster response symbol detection system using YOLOv8 for real-time object detection. The system supports both live webcam streaming and static image upload with GPS EXIF metadata for accurate geographic mapping.

## Architecture

### System Components

1. **Training Pipeline** (`training/`)
   - YOLOv8 model training on custom DroneAid symbol dataset
   - Dockerized environment with PyTorch 2.x
   - Outputs trained model to `training/models/droneaid/weights/best.pt`

2. **Inference API** (`inference/`)
   - FastAPI service on port 8000
   - Endpoints: `/detect` (multipart/form-data), `/health`
   - Accepts `conf_threshold` parameter (default 0.5)
   - Returns detections with bbox coordinates, class names, confidence scores

3. **Web Application** (`webapp/`)
   - React + TypeScript + Vite
   - Carbon Design System for UI components
   - Mapbox GL JS for mapping (token in `.env` as VITE_MAPBOX_ACCESS_TOKEN)
   - Dual input modes: webcam stream and image upload

### Key Technologies

- **ML**: YOLOv8 (Ultralytics), PyTorch 2.x
- **Backend**: FastAPI, Python 3.11
- **Frontend**: React 18, TypeScript, Vite 6
- **UI**: Carbon Design System (@carbon/react)
- **Mapping**: Mapbox GL JS 3.1.2
- **GPS**: exifr 7.1.3 for EXIF metadata extraction
- **Styling**: SCSS with CSS animations
- **Container**: Docker + Docker Compose

## Code Patterns & Conventions

### React State Management

- Use `useRef` for values that need to persist across render cycles (webcam status, processing flags)
- Sync refs with state using `useEffect` when values are checked in requestAnimationFrame loops
- Example:
  ```tsx
  const isWebcamActiveRef = useRef<boolean>(false);
  const [isWebcamActive, setIsWebcamActive] = useState(false);
  
  useEffect(() => {
    isWebcamActiveRef.current = isWebcamActive;
  }, [isWebcamActive]);
  ```

### Detection Interface

Always include location and timestamp for proper tracking:

```tsx
interface Detection {
  class_name: string;
  confidence: number;
  bbox: [number, number, number, number]; // [x, y, width, height]
  timestamp?: number;
  location?: [number, number]; // [longitude, latitude]
}
```

### Marker Lifecycle

- Markers fade in (0.5s), display for 30 seconds, then fade out (0.5s)
- Use timestamp-based keys to avoid re-processing same detections
- Clean up markers and timeouts on unmount

### API Integration

- Webcam: POST to `/api/detect` with FormData containing blob and conf_threshold
- Image Upload: POST to `/api/detect` with FormData containing file and conf_threshold
- Always include confidence threshold in API calls

## Important Configuration

### Map Settings

- Center: Puerto Rico (18.2208, -66.5901)
- Default zoom: 8 (shows full island)
- Style: 'mapbox://styles/mapbox/dark-v11'
- Markers: Custom 64x80px PNG with bottom anchor

### Detection Settings

- Webcam detection rate: 1 per second (1000ms throttle)
- Default confidence threshold: 0.6 (60%)
- Marker lifetime: 30,000ms (30 seconds)
- Fade animation duration: 500ms

### Symbols

8 DroneAid symbols with specific colors (extracted from icon PNGs):

- SOS: #ff6c00 (orange)
- OK: #00ce08 (green)
- Water: #418fde (blue)
- Food: #e22b00 (red-orange)
- Shelter: #00cbb3 (teal)
- First Aid: #ffed10 (yellow)
- Children: #cf8ffd (light purple)
- Elderly: #8c07ff (purple)

## Common Tasks

### Adding New Features

1. Update TypeScript interfaces in components
2. Add state management with refs for render loop values
3. Update SCSS with animations if needed
4. Test with both webcam and image upload modes
5. Update documentation in README, GETTING-STARTED, PROJECT-SUMMARY

### Modifying Detection Behavior

- Webcam throttle: Change `lastPredictionRef` comparison in App.tsx renderFrame
- Marker lifetime: Change setTimeout duration in DetectionMap.tsx
- Fade animations: Update CSS keyframes in DetectionMap.scss and App.scss

### GPS and Mapping

- GPS extraction uses exifr.gps() on File objects
- Real GPS: `detection.location = [longitude, latitude]` from EXIF
- Simulated: Random offset from Puerto Rico center for webcam
- Popup shows "GPS: lat, lng" for real coords, "Simulated location" otherwise

## File Structure Reference

```
droneaid-2026/
├── assets/
│   ├── images/          # Logo, icons (PNG)
│   └── markers/         # Custom marker images (64x80px)
├── training/
│   ├── Dockerfile       # Training container
│   ├── train.py         # YOLOv8 training script
│   └── models/          # Output directory
├── inference/
│   ├── Dockerfile       # Inference API container
│   ├── main.py          # FastAPI application
│   ├── model.py         # YOLOv8 inference wrapper
│   └── requirements.txt
├── webapp/
│   ├── src/
│   │   ├── App.tsx      # Main component (dual mode, webcam, upload)
│   │   ├── App.scss     # Styles with animations
│   │   ├── constants.ts # SYMBOL_COLORS mapping
│   │   └── components/
│   │       ├── DetectionMap.tsx      # Mapbox component
│   │       ├── DetectionMap.scss     # Map styles
│   │       └── ImageUpload.tsx       # File upload with EXIF
│   ├── public/
│   │   ├── favicon.ico
│   │   └── assets/
│   │       └── markers/ # 8 marker PNGs
│   ├── package.json
│   ├── Dockerfile
│   └── .env.example
├── docker-compose.yml
└── quickstart.sh
```

## Debugging Tips

### Webcam Not Detecting

- Check `isWebcamActiveRef.current` is true in renderFrame
- Verify model loaded: `modelLoadedRef.current`
- Check throttle timing: `lastPredictionRef.current`
- Console should show API requests to `/api/detect`

### Image Upload Issues

- Verify GPS extraction: Console shows "GPS coordinates for [filename]"
- Check API response: Console shows "API response: { detections: [...] }"
- Ensure confidence threshold isn't too high (default 0.6)
- Verify image contains recognizable symbols

### Map Problems

- Mapbox token must be in webapp/.env as VITE_MAPBOX_ACCESS_TOKEN
- Markers need timestamp for stable keys: `detection.timestamp`
- Check `processedDetectionsRef` to see if detections are being skipped
- Verify GPS coordinates: `detection.location[0]` (lng), `[1]` (lat)

### Animation Issues

- CSS transforms can conflict with Mapbox positioning - use opacity only for markers
- Detection list uses translateX for slide animations
- Ensure fade-in/fade-out classes are applied at correct lifecycle points
- Check setTimeout cleanup in useEffect return

## Development Workflow

1. Make changes to source files
2. Rebuild container: `docker compose build webapp` (or `inference`)
3. Restart service: `docker compose up -d webapp`
4. Check logs: `docker logs droneaid-2026-webapp-1` (or `-inference-1`)
5. Test in browser at http://localhost:3000

## Code Style

- TypeScript strict mode enabled
- Use functional components with hooks
- Descriptive variable names (e.g., `isWebcamActiveRef`, not `webcamRef`)
- Comments explain "why" not "what"
- SCSS nested selectors matching component structure
- Carbon Design System components for UI consistency

## Performance Considerations

- Webcam throttled to 1 FPS to reduce CPU/GPU load
- Marker cleanup with timeouts to prevent memory leaks
- RequestAnimationFrame for smooth video rendering
- Image uploads processed asynchronously
- Detection list and map updates use React state efficiently

## Security Notes

- Mapbox token in environment variables (not committed)
- CORS configured in FastAPI for localhost development
- Image uploads validated by FastAPI File type
- No sensitive data in localStorage or cookies

## Future Enhancements (from ROADMAP.md)

- Heat maps and clustering for detection patterns
- Real drone integration (DJI Tello/Mavic)
- Multi-user collaboration with database
- PWA for mobile deployment
- Enhanced ML with night vision and active learning
- REST API for external integration
- Redis caching and CDN for scalability

## Common Gotchas

1. **State in render loops**: Always use refs, not state, for values checked in requestAnimationFrame
2. **Marker key generation**: Must use stable keys (timestamp) or markers get re-processed every render
3. **CSS transforms on markers**: Conflicts with Mapbox positioning - use opacity animations only
4. **API endpoint**: It's `/api/detect` not `/api/predict`
5. **GPS coordinate order**: [longitude, latitude] not [lat, lng]
6. **Confidence threshold**: Sent as string in FormData, converted to float in API
7. **Fade animations**: Need forwards fill-mode to keep final state after animation
8. **Docker builds**: Changes to dependencies require full rebuild, not just restart

## Testing Checklist

- [ ] Webcam starts and shows video feed
- [ ] Detection starts automatically (no toggle)
- [ ] Bounding boxes drawn on webcam feed
- [ ] Detections appear in list with colors
- [ ] Markers appear on map (simulated locations)
- [ ] Markers fade in, stay 30s, fade out
- [ ] Image upload button works
- [ ] Multiple images can be uploaded
- [ ] GPS coordinates extracted from EXIF
- [ ] Images with GPS plot at correct map location
- [ ] Switching between modes works (webcam ↔ upload)
- [ ] Confidence slider adjusts threshold
- [ ] Map controls work (zoom, pan)
- [ ] Popups show detection details
- [ ] No console errors or warnings
- [ ] Favicon displays correctly

---
> Source: [Call-for-Code/DroneAid](https://github.com/Call-for-Code/DroneAid) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-24 -->
