<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Size Suggestion System - Copilot Instructions

This is a Python project for a shirt size recommendation system using computer vision and machine learning.

## Project Overview
- **Goal**: Recommend shirt sizes from body measurements captured via webcam/camera
- **Target Accuracy**: 96% minimum
- **Key Technologies**: OpenCV, MediaPipe, scikit-learn, TensorFlow

## Code Guidelines
1. Use type hints for all function parameters and return values
2. Follow PEP 8 style guidelines
3. Add comprehensive docstrings for all classes and functions
4. Implement proper error handling for camera operations
5. Use logging for debugging and monitoring
6. Structure code with clear separation of concerns:
   - Data collection (camera/image processing)
   - Feature extraction (body measurements)
   - Model training and prediction
   - UI/interface components

## Key Components
- **Body measurement extraction** using MediaPipe pose detection
- **Size chart mapping** based on Indonesian shirt sizing standards
- **Machine learning model** for size prediction
- **Real-time camera processing** with OpenCV
- **Calibration system** for accurate measurements

## Important Notes
- All measurements should be in centimeters
- Camera positioning and distance calibration is critical for accuracy
- Consider lighting conditions and pose standardization
- Implement confidence scoring for predictions

---
> Source: [Ichsannuur/size_suggestion_system](https://github.com/Ichsannuur/size_suggestion_system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-13 -->
