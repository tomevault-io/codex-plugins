# Copilot Instructions for PHOBos

## Project Purpose

**PHOBos** is a Python package for controlling and simulating optical test bench equipment in a kernel-nulling interferometry setup.

## CRITICAL CODING STANDARDS

### Language Requirements
⚠️ **ALL code, documentation, comments, docstrings, and user-facing text MUST be in ENGLISH.**
- No French text allowed in any part of the codebase
- This includes: code comments, docstrings, error messages, print statements, variable names, documentation files
- Exception: Historical commit messages may remain in French

### Documentation Standards
- Use NumPy-style docstrings for all functions and classes
- Include type hints where applicable
- Provide clear examples in docstrings for complex functions
- Keep README files and documentation updated

### Core Architecture Patterns

**Sandbox Mode Strategy**: The system implements a dual-mode architecture for development/testing without hardware:
- `SANDBOX_MODE = True` when BMC library is unavailable → uses mock classes from `src/phobos/sandbox/`
- `SANDBOX_MODE = False` for production → uses real hardware drivers
- All output prefixed with `⛱️ [SANDBOX]` in mock mode

**Hardware Classes**: Located in `src/phobos/classes/`, each inherits common patterns:
- `DM` (deformable_mirror.py): 169-segment hexagonal mirror control via BMC SDK
- `FilterWheel` (filter_wheel.py): 6-position Thorlabs wheel via serial
- `PupilMask` (pupil_mask.py): Combined Newport rotary + dual Zaber linear stages

**Configuration System**: 
- YAML/JSON config files for hardware ports and calibration data
- User config in `~/.kbch.json`, device configs like `DM_config.json`
- CLI tool `kbch` for interactive hardware control

## Development Workflow

### Environment Setup
```bash
# Conda is the preferred package manager for scientific environments
conda create -n phobos python=3.12
conda activate phobos
# Install package in development mode
pip install -e .
```

### Testing and Execution
⚠️ **CRITICAL: Always activate the virtual environment before running tests or Python commands**
- This project uses a `.venv` virtual environment located at the repository root
- Before running ANY Python code or tests, ALWAYS use: `D:/PHOBos/.venv/Scripts/python.exe`
- Never use system Python or `python` command directly
- This ensures correct dependencies and package versions are used

**Correct test execution:**
```bash
# Windows PowerShell
D:/PHOBos/.venv/Scripts/python.exe test_script.py
D:/PHOBos/.venv/Scripts/python.exe -c "import phobos; ..."
```

**Incorrect (DO NOT USE):**
```bash
python test_script.py  # ❌ Wrong - uses system Python
```

### Working in Sandbox Mode
When BMC SDK is not installed, the system automatically enters sandbox mode. All mock classes print debug information and simulate hardware responses. This is the default for development.

### Hardware Integration Points
- **BMC SDK**: Proprietary deformable mirror control (Windows/Linux)
- **Serial Communications**: PySerial for filter wheel and motion controllers
- **Config Loading**: PyYAML for configuration management

### Critical Dependencies
- `numpy`: Array operations for 169-segment DM control
- `pyserial`: All motor and filter communications
- `pyyaml`: Configuration file parsing

## Project-Specific Conventions

### Error Handling
- Hardware availability checked at import time in `__init__.py`
- Graceful degradation to sandbox mode with clear user feedback
- Serial timeouts and connection validation in all hardware classes

### Configuration Management
- Use absolute paths for device ports (e.g., `/dev/ttyUSB0`)
- Segment arrays are 0-indexed (0-168 for 169 segments)
- Position coordinates: piston (nm), tip/tilt (radians)

### CLI Design Patterns
```bash
kbch mask set 3           # Set mask position
kbch filter get           # Query current state  
kbch config mask add "name"  # Save current position
```

## Key Files for Understanding

- `src/phobos/__init__.py`: Mode detection and main imports
- `src/phobos/classes/deformable_mirror.py`: 169-segment control patterns
- `scripts/cli_linux.py`: Complete CLI interface implementation
- `DM_config.json`: Example of 169-segment calibration data
- `config.yml`: Hardware port mapping template

## Testing and Validation

Run in sandbox mode by default (no hardware required). For hardware testing, ensure:
1. BMC SDK installed for deformable mirror
2. Correct serial port permissions and device connections
3. Configuration files match actual hardware setup

The `timeit_tests.ipynb` notebook contains performance benchmarks for segment update operations.

## Common Debugging Steps

1. Check `SANDBOX_MODE` status at startup
2. Verify serial port permissions: `ls -la /dev/ttyUSB*`
3. Test configuration loading: `kbch config --show`
4. Monitor sandbox output for command simulation

## Atmospheric Simulation Module

### Overview
Located in `src/phobos/modules/atmosphere.py`, this module provides Kolmogorov-Von Karman atmospheric turbulence simulation based on XAOsim.

### Key Features
- **Phase screen generation**: Kolmogorov-Von Karman power spectrum
- **Frozen flow model**: Atmospheric turbulence evolution with wind
- **Multi-telescope support**: Calculate OPD for multiple apertures
- **Demo mode**: Real-time animation with matplotlib

### Usage Pattern
```python
from phobos.modules.atmosphere import get_delays

# Generate atmospheric phase delays
delays, times = get_delays(
    n_telescopes=4,
    r0=0.16,           # Fried parameter @ 500nm
    wind_speed=10.0,   # m/s
    demo=True          # Show animation
)
```

### Coding Conventions for Simulations
- Use SI units in function signatures (meters, seconds, radians)
- Return results in convenient units (nanometers for OPD)
- Provide sensible defaults for all parameters
- Include `demo` parameter for visualization when applicable

## Testing Requirements

### For Hardware Control
- Mock all hardware interactions in tests
- Use sandbox mode for CI/CD
- Test configuration loading separately from hardware

### For Simulations
- Validate physical scaling laws (e.g., r0 vs wavelength)
- Check statistical properties of generated data
- Test with multiple parameter combinations
- Verify output shapes and units

## Code Review Checklist

- [ ] All text is in English
- [ ] Docstrings follow NumPy style
- [ ] Type hints provided where applicable
- [ ] Examples included for complex functions
- [ ] Tests pass in sandbox mode
- [ ] Configuration files validated
- [ ] Error messages are clear and actionable

## Backward Compatibility
When refactoring or renaming classes, always ensure backward compatibility by keeping the old names as aliases with deprecation warnings.

Example:
```python
class OldName(NewName):
    def __init__(self, *args, **kwargs):
        warnings.warn("OldName is deprecated, use NewName instead", DeprecationWarning, stacklevel=2)
        super().__init__(*args, **kwargs)
```

## Documentation
Always document deprecated items in the code using docstrings and comments. Update the user documentation to reflect the changes but mention the deprecated items if they are still in use.

---
> Source: [VForiel/PHOBos](https://github.com/VForiel/PHOBos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-15 -->
