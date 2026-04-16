# AI Coding Agent Instructions for Hydrogel-Data-Analysis

## Project Overview
**Hydrogel-Data-Analysis** is a Python image analysis toolkit for hydrogel particle dynamics research. It combines TIFF image loading with particle tracking (trackpy) and multiple analysis methods to characterize particle motion in hydrogel and aqueous systems.

### Core Domains
- **Image I/O**: Load TIFF files with OME, Leica, and PCO metadata extraction
- **Metadata Management**: Standardize instrument metadata into unified `DatasetMetadata` objects
- **Particle Tracking**: Use trackpy for spot detection and trajectory linking; also imports TrackMate XML
- **Dynamics Analysis**: Three methods — MSD (mean-squared displacement), step size distributions, and SEM particle sizing
- **UI/Visualization**: Interactive parameter tuner (matplotlib-based) and SEM particle viewer (Qt-based)

## Critical Architecture Patterns

### 1. Data Pipeline: LoadedDataset → Analysis
```
TIFF → DatasetLoader → LoadedDataset (data + DatasetMetadata)
                    ↓
              Three analysis paths:
              1. trackpy_msd.main() - Spot detection → MSD
              2. MSD_Trackpy_clean.py - TrackMate XML → MSD  
              3. Schrittweiten_methode_*.py - TrackMate XML → Step size D
```

**Key Files**:
- [data_loader.py](hydro_analysis/data_loader.py): `DatasetLoader` class handles TIFF parsing
- [metadata.py](hydro_analysis/metadata.py): `DatasetMetadata` dataclass + utilities
- [trackpy_msd.py](hydro_analysis/trackpy_msd.py): Main MSD computation with spot detection
- [MSD_Trackpy_clean.py](hydro_analysis/MSD_Trackpy_clean.py): Cleaner XML→MSD pipeline
- [Particle_Parameter_Tuner.py](hydro_analysis/Particle_Parameter_Tuner.py): Interactive GUI for tuning detection params

### 2. Metadata Extraction Strategy
The loader applies **cascading metadata sources**:
1. Basic TIFF tags (Artist, Software, DateTime)
2. OME XML (pixel size, timestamps, channel names) — supports multiple OME versions
3. Leica-specific tags (stage position, Z-step)
4. PCO image metadata
5. Sidecar XML files from Data directory (pytrackmate outputs)

Each source fills missing fields; later sources don't overwrite. Example: `_populate_metadata_from_ome()` → `_populate_metadata_from_leica()` → `_populate_metadata_from_standard_tags()` → `_populate_metadata_from_sidecars()`.

**Convention**: Always preserve raw XML as `DatasetMetadata.raw_metadata` for debugging.

### 3. Dataset Kind Classification
`DatasetMetadata.infer_kind()` heuristically labels data as "FRAP", "SPT", or "FULL" based on:
- Keywords in notes + software field (case-insensitive)
- Axis counts: FRAP/bleach → "FRAP"; sparse C + T>100 → "SPT"; Z>1 + T>1 → "FULL"

This hints at analysis strategy without requiring explicit user input.

### 4. Trackpy Integration & MSD Computation
**Parameters** (from `trackpy_msd.main()`):
- `diameter`, `distance`, `minmass`: Spot detection (trackpy.locate)
- `mpp` (µm/px), `fps`: Calibration (from metadata)
- `smooth`, `radius`: Optional background subtraction (Gaussian or rolling-ball)

**Output**:
- Trajectories as pandas DataFrame with columns: `particle`, `frame`, `x`, `y`
- MSD computed via trackpy's built-in; power-law fitting via `fit_powerlaw_with_errors()`

**Files**:
- [MSD_Trackpy_clean.py](hydro_analysis/MSD_Trackpy_clean.py): XML parsing with `load_tracks_xml()`; outputs trackpy-compatible DataFrame
- [pytrackmate_MSD_XML.py](hydro_analysis/pytrackmate_MSD_XML.py): Batch analysis over multiple TrackMate XMLs; includes particle size detection
- [trackpy_msd.py](hydro_analysis/trackpy_msd.py): End-to-end: TIFF → spot detection → linking → MSD
- [MSD_Trackpy.py](hydro_analysis/MSD_Trackpy.py): Older version; legacy

### 5. Interactive Parameter Tuning
[Particle_Parameter_Tuner.py](hydro_analysis/Particle_Parameter_Tuner.py) provides a matplotlib-based GUI for optimizing trackpy parameters:
- **Real-time preview**: Shows detected spots overlaid on selected frame
- **Live linking**: Displays particle trajectories as they're linked
- **Adjustable params**: Sliders for diameter, minmass, search_range, memory, rolling-ball/Gaussian filters
- **Frame navigation**: Step through video to verify detection quality
- **Export tracks**: Save linked DataFrame to pickle when satisfied

**Usage Pattern**: Run standalone on a TIFF to find optimal parameters before batch processing.

### 6. SEM Particle Analysis
Two complementary tools for analyzing scanning electron microscopy images of hydrogel particles:
- [sem_particle_analysis.py](hydro_analysis/sem_particle_analysis.py): Core analysis logic (watershed segmentation, diameter measurement)
- [sem_particle_viewer.py](hydro_analysis/sem_particle_viewer.py): Qt GUI with histogram display, overlay visualization, and parameter adjustment

## Developer Workflows

### Running Scripts Directly
Most scripts are standalone modules with `main()` functions:
```bash
# MSD analysis from TIFF
python -m hydro_analysis.trackpy_msd

# Interactive parameter tuning
python hydro_analysis/Particle_Parameter_Tuner.py

# Step size diffusion analysis (batch)
python hydro_analysis/Schrittweiten_methode_D0.py
```

### Loading a Dataset Interactively
```python
from hydro_analysis.data_loader import DatasetLoader
from pathlib import Path

path = Path("Data/2025_09_01_16_03_50--FRAP/FRAP 006/FRAP Pb1 Series24/*.tif")
loader = DatasetLoader(path)
loaded = loader.load()
print(loaded.metadata.infer_kind())
print(loaded.metadata.as_dict())
```

### Running MSD Analysis
```python
from hydro_analysis.trackpy_msd import main

main(
    tif_path="Data/.../image.tif",
    diameter=9, distance=10, minmass=420,
    mpp=0.065, fps=1.0,  # Get from metadata
    plot=True, smooth=True, radius=50
)
```

### Running Parameter Tuner
```bash
# Launch interactive GUI for parameter optimization
python hydro_analysis/Particle_Parameter_Tuner.py
# Select TIFF file via dialog, adjust sliders in real-time
# Export linked tracks via "Export Tracks" button
```

## Code Conventions

### 1. Dataclasses & Type Hints
All metadata is stored in `@dataclass` objects (e.g., `DatasetMetadata`, `TimeSummary`). Always use type hints; default to `Optional[Type]` for nullable fields.

**Example**:
```python
@dataclass
class DatasetMetadata:
    path: Path
    px_size_xy_um: Optional[float] = None
    timestamps: List[float] = field(default_factory=list)
```

### 2. Axes Convention
- Axes string follows ImageIO/OME: "CZYX", "TCZYX", etc.
- Map axes to dimensions via `dict(zip(metadata.axes, metadata.shape))`.
- X, Y are spatial; Z is depth; T is time; C is channel.

**Example from metadata.py**:
```python
@property
def width_height(self) -> Tuple[int, int]:
    axes_to_dim = dict(zip(self.axes, self.shape))
    return axes_to_dim.get("X", 0), axes_to_dim.get("Y", 0)
```

### 3. Error Handling
- XML parsing: Wrap in try-except; return silently if malformed (don't fail the whole load).
- File I/O: Raise `FileNotFoundError` early; use `Path.exists()` checks.
- Metadata defaults: Always provide sensible fallbacks (e.g., `dt = 1.0` if frameInterval absent).

### 4. Calibration from Metadata
- Pixel size from OME `PhysicalSizeX/Y` (µm/px); fallback to Leica tags.
- Z-step from OME `PhysicalSizeZ` or Leica ScanInfo.
- Time from OME `Plane.DeltaT` or root `frameInterval` (seconds).
- Stage position (mm) from Leica acquisition metadata.

### 5. Trackpy DataFrame Schema
After `load_tracks_xml()` or spot detection:
```
columns: ['particle', 'frame', 'x', 'y']
attrs:
  'mpp': float (µm/px)
  'fps': float (frames/second)
```

Trackpy's MSD functions rely on these attributes; always set them.

## Integration Points

### External Data: PyTrackMate XML
[pytrackmate_MSD_XML.py](pytrackmate_MSD_XML.py) parses TrackMate outputs with frame interval and spatial unit metadata. Converts to trackpy-compatible DataFrame.

### PCO Image Files
[frequency_check_Pco.py](hydro_analysis/frequency_check_Pco.py) reads PCO-specific TIFF tags. Integrated into `_populate_metadata_from_standard_tags()`.

### Step Size Diffusion Analysis
- [Stepsize.py](hydro_analysis/Stepsize.py): Core step size (displacement) calculation utilities
- [Schrittweiten_methode_D0.py](hydro_analysis/Schrittweiten_methode_D0.py): Batch processing of TrackMate XML for water measurements
- [Schrittweiten_methode_20mg.py](hydro_analysis/Schrittweiten_methode_20mg.py): Analysis for 20mg/ml hydrogel samples

**Method**: Calculates diffusion coefficient from frame-to-frame displacement distributions (dx, dy) by fitting Gaussians and using D = σ² / (2*dt).

## Common Pitfalls

1. **OME Namespace**: Schemas vary (2015-01, 2016-06). Always check both; don't assume fixed URI.
2. **Background Subtraction**: Rolling-ball radius must match (2×radius) not radius alone for cv2 kernels.
3. **Frame Indexing**: XML frame indices are 0-based; ensure consistency with pandas DataFrames.
4. **Missing Metadata**: Gracefully handle absent calibration; infer_kind() uses heuristics, not guarantees.
5. **Parameter Tuning**: Always use Particle_Parameter_Tuner.py before batch processing to find optimal trackpy parameters for your specific data.

## File Organization
```
hydro_analysis/
  __init__.py              # Module exports (DatasetLoader, DatasetMetadata)
  data_loader.py           # TIFF I/O & metadata extraction
  metadata.py              # DatasetMetadata + utilities
  trackpy_msd.py           # Main MSD analysis pipeline
  MSD_Trackpy_clean.py     # Cleaner trackpy wrapper (prefer)
  pytrackmate_MSD_XML.py   # TrackMate XML parsing
  Particle_Parameter_Tuner.py  # Interactive parameter UI
  frequency_check_Pco.py   # PCO-specific metadata
  Stepsize.py              # Step/displacement analysis
  Schrittweiten_methode_D0.py    # Batch step size analysis (water)
  Schrittweiten_methode_20mg.py  # Batch step size analysis (20mg/ml)
  sem_particle_analysis.py       # SEM particle core analysis
  sem_particle_viewer.py         # Qt-based SEM GUI
```

Data stored in `Data/` with instrument-dated subdirectories (e.g., `2025_09_01_16_03_50--FRAP/`).

## Refactoring Guidelines

**IMPORTANT**: This codebase is undergoing active refactoring. See [REFACTORING_PLAN.md](../REFACTORING_PLAN.md) for the comprehensive restructuring strategy.

### Current Refactoring Goals
1. **Consolidate Duplicate Code**: Functions like `fit_powerlaw_with_errors()`, `load_tracks_xml()`, and `subtract_background()` are duplicated across 4-7 files
2. **Create Core Modules**: Centralized `core/` directory for io, analysis, tracking, visualization
3. **Flexible Processing**: Support single-file, batch, and folder processing modes automatically
4. **Parallelization**: Enable multi-threaded processing for independent files
5. **Structured Logging**: Replace print statements with proper logging (especially for batch operations)
6. **Deferred Visualization**: Collect results first, then aggregate and save figures (not immediate saving)

### When Adding New Analysis Code
- **Don't duplicate**: Check if similar functions exist in other scripts first
- **Use core modules**: If refactored `core/` modules exist, import from there
- **Follow workflow pattern**: Inherit from `BaseWorkflow` if implementing new analysis types
- **Log, don't print**: Use `logging` module instead of print() for batch processing
- **Return figures**: Visualization functions should return `plt.Figure` objects, not save directly
- **Support flexibility**: Design functions to work with single files or lists of files
- **Comments**: Comments and Print statements should be as few as possible but with necessary decision making explanations. No emojis, no exclamation marks. Formal tone only.

### Migration Status
Check [REFACTORING_PLAN.md](../REFACTORING_PLAN.md) for current phase and which modules have been refactored.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Qindral) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
