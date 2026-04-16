<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Blob Tracker Project Instructions

## Project Overview
This is a Python-based blob detection and tracking system using OpenCV. The project detects colored blobs in video streams and generates directional commands for rover navigation.

## Technology Stack
- **Language**: Python 3.7+
- **Main Libraries**: 
  - OpenCV (cv2) - Computer vision and image processing
  - NumPy - Numerical computations
- **Purpose**: Real-time blob detection and tracking with interactive HSV tuning

## Key Components
- **BlobTracker class**: Main tracking engine with HSV-based detection
- **Detection pipeline**: HSV conversion → Masking → Morphological operations → Contour detection
- **Tracking state**: Maintains last position and frame loss counter for robustness
- **Interactive UI**: Trackbars for real-time parameter adjustment

## Development Guidelines
1. Maintain the BlobTracker class structure for modularity
2. Keep image processing pipeline efficient for real-time performance
3. Ensure trackbar values are read every frame for responsive tuning
4. Handle camera initialization gracefully with try-except blocks
5. Document all HSV range modifications with comments

## Testing Considerations
- Test with various lighting conditions
- Validate blob detection with different color objects
- Verify tracking performance with moving objects
- Check frame loss recovery mechanism

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Subodh584) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
