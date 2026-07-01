# Copilot Instructions for picture-rotation-fixer

## Project Overview

This is a Python tool that automatically rotates scanned family photos to the correct orientation using intelligent cascading detection. The tool uses a multi-tier approach: face detection first (OpenCV Haar cascades), then object detection fallback (YOLOv8/YOLO11), maximizing the number of images that can be automatically corrected.

## Architecture & Components

### Core Functionality
- **Main Script**: `rotate.py` - Contains all image processing logic
- **Cascading Detection Logic**: 
  1. Face detection using OpenCV Haar cascades (fast, accurate for portraits)
  2. Object detection using YOLOv8/YOLO11 OBB (Oriented Bounding Box) for general objects
  3. Leave unchanged if no content detected
- **Batch Processing**: Multiprocessing with progress bars for performance
- **File Handling**: Recursive directory scanning, supports PNG/JPG/JPEG

### Key Dependencies
- `opencv-python`: Face detection via Haar cascades
- `ultralytics`: YOLOv8/YOLO11 object detection with OBB support
- `pillow`: Image manipulation and rotation
- `tqdm`: Progress bars for batch operations
- `pytest`: Testing framework with mocking

### Distribution Methods
- **Homebrew Formula**: macOS package manager integration
- **Standalone Binaries**: Cross-platform executables (Linux/macOS/Windows) with bundled models
- **Python Package**: Direct installation via pip/uv

## Development Guidelines

### Code Style & Standards
- Follow PEP 8 Python conventions
- Use type hints where appropriate
- Keep functions focused and single-purpose
- Prefer explicit over implicit behavior
- Use descriptive variable names that indicate content (e.g., `faces_detected`, `objects_found`)

### Error Handling
- Gracefully handle missing files or corrupted images
- Provide informative error messages to users
- Continue processing other images if one fails
- Log warnings for images that couldn't be processed

### Performance Considerations
- Use multiprocessing for batch operations
- Implement lazy loading for YOLO models (only load when needed)
- Cache detection results where possible
- Optimize for memory usage with large image batches

### Testing Strategy
- **Unit Tests**: Mock external dependencies (YOLO models, OpenCV)
- **Integration Tests**: Test with sample images
- **Edge Cases**: Test with corrupted images, empty directories, various image formats
- **Performance Tests**: Verify multiprocessing works correctly

## File Structure & Key Files

```
├── rotate.py                 # Main application script
├── pyproject.toml            # Project dependencies and configuration
├── tests/
│   └── test_enhanced.py      # Comprehensive pytest suite
├── .github/
│   └── workflows/
│       └── build-and-release.yaml  # CI/CD for cross-platform binaries
├── README.md                 # User documentation
└── LICENSE                   # MIT license
```

## Common Tasks & Patterns

### Adding New Detection Methods
1. Implement detection function that takes image and returns confidence/bounding boxes
2. Add to cascading logic in `auto_rotate()` function
3. Update tests with appropriate mocking
4. Document new capabilities in README

### Modifying Image Processing
- Always preserve original image if no rotation needed
- Maintain EXIF data when possible
- Handle different color spaces (RGB, BGR, RGBA)
- Test with various image sizes and formats

### Updating Dependencies
- Test compatibility with both standalone binary and Python package modes
- Verify PyInstaller bundling works with new dependencies
- Update GitHub Actions workflow if new data files need bundling
- Run full test suite across all platforms

## CI/CD & Distribution

### GitHub Actions Workflow
- **Cross-Platform Builds**: Linux, macOS (x64/ARM64), Windows
- **YOLO Model Bundling**: Pre-downloads models and bundles with executables
- **Conditional Bundling**: Gracefully handles cases where models can't be downloaded
- **Artifact Management**: Automatic versioning and release creation

### Binary Distribution
- Uses PyInstaller for standalone executable creation
- Bundles OpenCV Haar cascade data and YOLO models
- Platform-specific data path handling (Windows vs Unix)
- Hidden imports for ultralytics submodules

## Known Issues & Gotchas

### YOLO Model Handling
- Models download to platform-specific cache directories
- PyInstaller requires explicit bundling of model files
- Fallback logic needed for environments where models can't be pre-downloaded
- Model paths vary between platforms (Linux: `~/.ultralytics/`, macOS: `~/Library/Application Support/`, Windows: `~/AppData/Roaming/`)

### Image Format Considerations
- Some images may have incorrect EXIF orientation data
- Progressive JPEG files may require special handling
- Large images may consume significant memory during processing

### Multiprocessing Limitations
- Progress bars need special handling in multiprocessing context
- Shared state should be minimized between processes
- Error handling must account for subprocess failures

## Development Environment Setup

```bash
# Install dependencies
uv install

# Run tests
uv run pytest tests/

# Run with sample images
uv run python rotate.py /path/to/test/images

# Build standalone binary
uv run pyinstaller rotate.py --onefile --name picture-rotation-fixer
```

## Code Review Guidelines

### What to Look For
- Proper error handling for image processing operations
- Memory efficiency with large image batches
- Cross-platform compatibility (file paths, model loading)
- Test coverage for new detection methods
- Documentation updates for user-facing changes

### Testing Checklist
- [ ] Unit tests pass with mocked dependencies
- [ ] Integration tests work with real images
- [ ] Cross-platform binary builds succeed
- [ ] Performance acceptable with large image sets
- [ ] Memory usage remains reasonable

## User Experience Principles

### Design Goals
- **Zero Configuration**: Works out of the box with sensible defaults
- **Intelligent Fallbacks**: Gracefully handles edge cases
- **Clear Feedback**: Progress bars and informative messages
- **Non-Destructive**: Preserves originals when no rotation needed
- **Fast Processing**: Leverages multiprocessing for batch operations

### Error Messaging
- Use friendly, actionable error messages
- Provide context about what went wrong
- Suggest solutions when possible
- Continue processing other images when one fails

When contributing to this project, focus on maintaining the balance between accuracy, performance, and user experience. The cascading detection approach should always prioritize face detection for best results while providing object detection as a reliable fallback.

---
> Source: [JustinGuese/picture-rotation-fixer](https://github.com/JustinGuese/picture-rotation-fixer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-01 -->
