**Overview**
- **Purpose:** Python bindings over NBIS (NIST Biometric Image Software) for minutiae extraction, matching (Bozorth3), and quality (NFIQ).
- **Core Modules:** `pynbis/core.py` (Python API + dataclasses), `pynbis/_nbis_ext.c` (C extension wrapping NBIS), `setup.py` (compiles NBIS from `nbis_src/`).
- **Data Model:** Images are `np.uint8` grayscale; 2D arrays or 3D with 1 channel only.

**Architecture**
- **Flow:** Python validates inputs → C extension runs NBIS → Python returns typed results. Binarized image from MINDTCT is cached on `Fingerprint`.
- **NBIS Sources:** Compiled directly from `nbis_src/**` (bozorth3, mindtct, nfiq, commonnbis, imgtools/wsq+jpegl). Do not modify NBIS sources unless adding missing deps.
- **Extension API:** `_nbis_ext.extract_minutiae(image, ppi)`, `_nbis_ext.match_fingerprints(probe, gallery)`, `_nbis_ext.compute_nfiq(image, ppi)`, `_nbis_ext.match_xyt(probe_list, gallery_list)`, `_nbis_ext.decode_wsq(wsq_bytes)`.
- **Python API:** See `pynbis/__init__.py` exports: `extract_minutiae`, `match_fingerprints`, `compute_quality`, `match_minutiae`, `Fingerprint`, and `decode_wsq`.

**Build & Test**
- **Dev install (Linux):** ensure build tools, then install to compile the C extension.
```bash
sudo apt-get update && sudo apt-get install -y build-essential python3-dev
pip install -e .[dev]
```
- **Run tests:** `pytest -q` (unit tests do not require the extension; examples do).
- **Examples:** run from `examples/` after a successful build, e.g. `python examples/02_fingerprint_matching.py`.
- **Packaging:** `python -m build` produces wheels/sdist (requires `pip install build`).

**Conventions & Patterns**
- **Inputs:** Always accept `np.ndarray` of dtype `uint8`; reject color images (enforced in `Fingerprint.__init__`).
- **Types:** Use `dataclasses` (`Minutia`, `MatchResult`, `QualityResult`) and `numpy.typing` hints; keep public API re-exported in `pynbis/__init__.py` and `__all__` updated.
- **Thresholding:** `Fingerprint.match(..., threshold)` sets `matched` only when a threshold is passed; keep this semantics in wrappers.
- **Utils:** Image I/O and helpers live in `pynbis/utils.py`. Extras are behind `[imaging]` (Pillow, scipy, opencv-python). Do not add hard deps to core.

**Working With the C Extension**
- **Add new capability:**
  - Expose a C function in `pynbis/_nbis_ext.c` (add to `NBISMethods`), using `PyArray_FROM_OTF` for safe `uint8` input conversion.
  - Wrap it in `pynbis/core.py` with input validation and typed results; surface via `pynbis/__init__.py`.
  - If NBIS code requires additional C files, extend `get_nbis_sources()` in `setup.py` and include dirs in `get_nbis_includes()`.
- **Do not:** change NBIS headers/APIs casually; prefer adapting the wrapper.

**Gotchas & Troubleshooting**
- **ImportError `_nbis_ext`:** the Python API will raise `RuntimeError("NBIS extension not available")`. Fix by compiling via `pip install -e .` and ensuring build toolchain is present.
- **Image shape/dtype:** only grayscale `uint8` arrays are valid; 3D arrays must have 1 channel.
- **Performance:** for 1:N, pre-extract minutiae and use `match_minutiae` to avoid repeated extraction.
- **Thresholds:** matching scores vary; repo examples and `README.md` document typical ranges and decision thresholds.

**File Map (start here)**
- `pynbis/core.py`: main Python API, validations, object model.
- `pynbis/_nbis_ext.c`: NBIS bindings exposed to Python.
- `setup.py`: lists NBIS sources/includes; controls compile flags.
- `nbis_src/`: vendored NBIS sources (bozorth3, mindtct, nfiq, commonnbis...).
- `tests/test_core.py`: minimal sanity tests for dataclasses and objects.
- `examples/*.py`: runnable usage patterns matching `README.md`.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rodfileto)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/rodfileto)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
