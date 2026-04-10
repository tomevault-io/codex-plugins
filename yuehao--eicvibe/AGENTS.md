# EICViBE AI Development Instructions

## 🏗️ Architecture Overview

EICViBE is an accelerator physics simulation framework with a **modular, inheritance-based** design:

- **Machine Portal** (`src/eicvibe/machine_portal/`): Core lattice modeling with prototype/instance pattern
- **Simulators** (`src/eicvibe/simulators/`): Physics engines (XSuite, future other simulation engine) with 3-mode simulation (LINAC/RING/RAMPING)
- **Models** (`src/eicvibe/models/`): Pydantic-validated parameter groups and base classes
- **Parameter Groups**: Physics-validated containers (MagneticMultipoleP, RFP, BendP) replacing generic dictionaries

## 📁 Project Structure

- `src/eicvibe/`: Core package
  - `machine_portal/`: Lattice, elements, parameter groups (YAML schemas in `elements.yaml`, `parameters.yaml`)
  - `simulators/`: Engine contracts and XSuite interface (`interface.yaml`, `registry.py`)
  - `models/`: Pydantic models and validators
  - `utilities/`: MAD-X import and helpers
- `tests/`: Pytest suites (`test_*.py`) for production code
- `docs/`: Sphinx + MyST documentation
- `agent/`: AI-generated artifacts (development docs, exploratory notebooks, demos)
- Root config: `pyproject.toml` (hatchling build), `uv.lock` (env), `README.md`

**All AI-generated middle-steps documents and tests should be only in `agent/` folder.**

## 🔧 Development Environment

Always use `uv` to manage dependencies and run scripts/tests:

```bash
uv sync                    # Install dependencies
uv run python script.py    # Run with managed environment
uv run pytest tests/       # Run tests
uv run sphinx-build -b html docs docs/_build/html  # Build docs
```

## 🎯 Development Patterns

### Element Creation & Parameter Management
```python
# Always use parameter groups, never direct parameter dictionaries
quad = Quadrupole(name="qf", length=0.5)
mag_params = MagneticMultipoleP(kn1=2.0)  # Pydantic validation
quad.add_parameter_group(mag_params)

# Access via parameter groups API
strength = element.get_parameter("MagneticMultipoleP", "kn1")
```

### Lattice Design Philosophy
- **Prototypes first**: Create element definitions with `inherit=None`
- **Instances inherit**: Use `add_element_to_branch()` for auto-naming (`prototype_{n}`)
- **Branch topology**: Specify `branch_type="ring"` or `"linac"` - affects selection behavior
- **Element selection**: Use rich filtering with `select_elements()` (position ranges, regex, relative positioning)

### Simulation Engine Contract
All engines should inherit from `BaseSimulationEngine` with abstract methods:
```python
initialize_engine() -> bool
convert_lattice(lattice, branch_name) -> Any
create_particles(particle_params) -> Any
track_single_turn() -> bool
get_particle_coordinates() -> Dict[str, np.ndarray]
update_element_parameter(element, group, param, value) -> bool
get_element_parameter(element, group, param) -> float
```

## 🧪 Testing Guidelines

- **Unit tests for CI**: `tests/test_*.py` for automated validation
- **XSuite integration tests**: Handle graceful fallback when XSuite unavailable (use `pytest.importorskip("xsuite")`)
- **Parameter validation tests**: Test Pydantic models with physics constraints
- **Exploratory notebooks**: `agent/tests/*.ipynb` for development/debugging (not in production tests)
- Run: `uv run pytest -q tests/` before pushing

## ⚠️ Critical Integration Points

### Parameter Unit System
- **EICViBE uses SI with radian**: RF phases, bend angles in radians
- **XSuite expects radians except RF cavity lag (in degree)**: Use `interface.yaml` conversion factors
- **Direct mappings**: RF voltage/frequency, magnetic strengths (no conversion)

### MAD-X Import Integration
```python
from eicvibe.utilities.madx_import import import_madx_lattice
lattice = import_madx_lattice("file.madx", consolidate_drifts=True)
```

### Simulation Modes
- **Ring**: Optics calculated with periodic boundary conditions; tracking with asynchronous BPM buffer retrieval
- **Linac**: Optics calculated with initial conditions; tracking start-to-end with BPM readings at the end
- **Ramping**: Simulate using ramping plan; optics calculated at selected time steps

## 🚨 Common Pitfalls

1. **Never bypass parameter groups**: Always use `element.get_parameter(group, param)` not `element.parameters[key]`
2. **Check simulation engine availability**: Use graceful fallback patterns
3. **Branch expansion**: Use `lattice.expand_lattice(force_expand=True)` after setting root branch
4. **Parameter validation**: Leverage Pydantic validators, don't write manual validation
5. **Unit conversions**: Use `interface.yaml` mappings for each engine, don't hardcode conversion factors

## 🎯 Code Quality Standards

- **Pydantic models**: Use for all parameter validation (models/parameter_groups.py patterns)
- **Type hints**: Required for all new code
- **Naming**: Classes `CamelCase`, functions/methods/variables `snake_case`, modules `snake_case.py`
- **Physics validation**: Implement via Pydantic field validators
- **Error handling**: Graceful degradation with clear error messages
- **Documentation**: Docstrings with physics context and units

## 📝 Commit Guidelines

- Commits: imperative mood and scoped, e.g., `machine_portal: validate kn1 via Pydantic`
- PRs: include clear description, linked issues, test coverage notes
- Keep changes focused; update docs when behavior changes

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yuehao)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/yuehao)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
