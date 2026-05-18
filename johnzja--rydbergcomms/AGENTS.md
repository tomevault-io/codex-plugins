# Copilot instructions for `rqr`

## Big picture
- This repository mixes MATLAB-first research code with Python ports and a Flask UI for Rydberg atomic receiver simulation.
- The canonical physics model lives in MATLAB scripts at the repo root, especially `configureRAQR.m`, `transientQuantumSimulation.m`, and helpers like `getNormalQuadrature.m` and `getPhysicalConstant.m`.
- The Python runtime mirrors that MATLAB flow in `python/utils/transient_quantum.py`; preserve parity with the MATLAB equations and indexing conventions when changing the simulator.
- The web app entry point is `app.py`. It generates IF input waveforms, runs one of two RK4 engines (`python` or `matlab`), and serves JSON to `templates/index.html` + `static/js/app.js`.

## Core data flow
- Web simulation flow in `app.py` is: generate IF waveform (`generate_gaussian_pulse()` / `generate_step_input()`) → upsample to RK rate in `prepare_rk_input()` → run engine → convert `probeResponse` back to real IF with `probe_response_to_if_waveform()` → serialize plots with `serialize_waveforms()`.
- The Python engine uses `TransientQuantumSimulator.run()` with `configure_raqr("Transit")`, `dt = 1e-9`, `Nd = 1501`, and `n_jobs=-1`.
- The MATLAB engine is bridged by `runTransientQuantumWeb.m`; Flask writes `.mat` inputs with `scipy.io.savemat`, calls `matlab -batch`, then reads `.mat` output back with `loadmat`.
- `python/main.py` is the best reference for the intended IF-level signal chain and plotting behavior; `compute_spectrum()` in `app.py` intentionally matches `showSpectrum()` there.

## Important directories
- Root `*.m` files: paper-grade simulation scripts and analytical models.
- `python/utils/`: Python quantum simulator, optional C++ RK4 kernel, and parity/benchmark tests.
- `python/utils/tests/`: validation of Python vs C++ RK4 (`test_rk4_cpp_parity*.py`) and timing scripts (`benchmark_rk4.py`, `run_bench_nd1501_nt1000.py`).
- `templates/` and `static/`: Flask frontend.
- `results/` and `wfsims/`: generated figures and waveform-level MATLAB experiments.

## Developer workflows
- Start the web app with `python app.py` from the repo root.
- Build the optional accelerated RK4 shared library with `make all`, which runs `python/utils/build_rk4.sh`.
- If the shared library exists, `app.py` auto-detects it in `python/utils/` and enables `use_cpp_rk=True` for the Python engine.
- Use the parity tests in `python/utils/tests/test_rk4_cpp_parity.py` and `test_rk4_cpp_parity_multithread.py` before modifying RK4 internals.
- Use `python/utils/tests/benchmark_rk4.py` when evaluating performance regressions; this codebase cares about runtime for large RK grids.

## Project-specific conventions
- Keep MATLAB and Python implementations behaviorally aligned; if you change one side of the simulator, inspect the corresponding file on the other side.
- Preserve scientific constants and default parameters exactly unless the change explicitly targets model calibration.
- Many numerical arrays follow MATLAB column-major conventions; `transient_quantum.py` uses `reshape(..., order='F')` for that reason.
- Trimming/resampling steps are part of the model pipeline, not cleanup: e.g. probe response trimming (`500`) and IF resample edge trimming (`50`) in `app.py` and `python/main.py`.
- Waveform plots in the UI expect complex IF input and real IF output; do not silently convert the receiver output to complex unless the UI contract changes.

## Integration notes
- MATLAB is an external runtime dependency for the `matlab` engine; `app.py` resolves it via `MATLAB_EXE` or `PATH`.
- Python dependencies are scientific-stack heavy (`numpy`, `scipy`, `flask`, `matplotlib`), and the simulator may also use multiprocessing plus the optional compiled RK4 DLL/SO.
- The OFDM helper in `python/utils/sim_OFDM.py` defines the standard baseband rate (`200e3`) that is upsampled to the IF rate (`2 MHz`) in current web flows.
- When editing UI behavior, keep API fields in sync across `app.py`, `templates/index.html`, and `static/js/app.js`.

---
> Source: [johnzja/RydbergComms](https://github.com/johnzja/RydbergComms) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->
