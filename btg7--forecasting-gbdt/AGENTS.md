1 . Hardware constraints (dev & initial prod)
Resource	Spec	Implication
CPU	AMD Ryzen 7 5800X (8 cores, 16 threads)	XGBoost/LightGBM tree_method='hist' finishes 1 M rows × 50 features × 500 iters in ≈1 min on a 4-core i7; our 8-core Ryzen easily trains overnight on datasets ≲ 10 M rows. 
xgboost.readthedocs.io
GPU	AMD Radeon RX 6650 XT	No CUDA → mainstream GPU wheels of XGBoost cannot run (require NVIDIA CC ≥ 5.0). 
xgboost.readthedocs.io
 LightGBM’s OpenCL backend can work with AMD but needs a custom compile and up-to-date OpenCL/ROCm driver, so we defer it. 
lightgbm.readthedocs.io
lightgbm.readthedocs.io

Decision ▶ CPU-only builds for now – simpler Docker image, reproducible, and fast enough for monthly retrains.

2 . Environment choices
Layer	Selected option	Rationale / commands
Python	3.11	Latest LTS-style version broadly supported by XGBoost ≥ 1.7.0. (Minor packaging quirks noted but resolved). 
github.com
Dependency manager	Poetry	Easy learning curve, lockfile for deterministic Docker builds, plays well with pip wheels.
Base image	python:3.11-slim-bullseye	Small, Debian-based, security-patched.
GBDT lib	xgboost-cpu >=2.0 (pip wheel)	No GPU pieces; install is a single pip step.
Aux	scikit-learn, pandas, numpy	Feature engineering & evaluation stack.

3 . Repository scaffold
text
Copy
Edit
forecasting_gbdt/
├── pyproject.toml
├── poetry.lock
├── Dockerfile
├── train.py
├── src/
│   └── forecasting/  # package code
└── tests/
3.1 pyproject.toml (excerpt)
toml
Copy
Edit
[tool.poetry]
name = "forecasting_gbdt"
version = "0.1.0"
description = "Gradient-boosting demand-forecasting service"
authors = ["Your Name <you@example.com>"]
readme = "README.md"

[tool.poetry.dependencies]
python = "^3.11"
xgboost-cpu = "^2.0"
scikit-learn = "^1.5"
pandas = "^3.0"
numpy = "^2.0"
Create & lock:

bash
Copy
Edit
poetry env use 3.11
poetry install --no-root
3.2 Dockerfile
Dockerfile
Copy
Edit
FROM python:3.11-slim-bullseye

# System deps for manylinux wheels
RUN apt-get update && apt-get install -y --no-install-recommends \
        build-essential git && \
    rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY pyproject.toml poetry.lock ./
RUN pip install --no-cache-dir poetry && poetry install --no-dev --no-root

COPY . .
CMD ["poetry", "run", "python", "train.py"]
4 . CI/CD blueprint (GitHub Actions)
yaml
Copy
Edit
name: CI

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-22.04
    strategy:
      matrix:
        python-version: ["3.11"]
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with: {python-version: ${{ matrix.python-version }}}
      - name: Install Poetry
        run: pip install poetry
      - name: Install deps
        run: poetry install --no-root
      - name: Run unit tests
        run: poetry run pytest -q

  docker:
    runs-on: ubuntu-22.04
    needs: test
    steps:
      - uses: actions/checkout@v4
      - name: Build & push image
        run: |
          docker build -t your-registry/forecasting_gbdt:${{ github.sha }} .
          echo "${{ secrets.DOCKER_TOKEN }}" | docker login -u your-user --password-stdin
          docker push your-registry/forecasting_gbdt:${{ github.sha }}
5 . Monthly retrain workflow
On-prem store install: first run train.py --full to train on the entire historical dataset.

Scheduled job (e.g., GitHub Actions cron or K8s CronJob) fetches last-month sales, appends to feature store, retrains, and pushes a fresh model artifact to S3 / your registry.

API deployment pulls the newest model.pkl at service start; inference remains CPU-only and lightweight.

6 . Upgrade roadmap
Trigger	Action
Data grows >30 M rows or retrain ≤2 h no longer met	Add NVIDIA GPU nodes; switch to xgboost CUDA wheels & set tree_method='gpu_hist'.
Need real-time inference at scale	Consider Treelite or ONNX export for ultra-fast CPU serving.
Want to exploit Radeon GPU	Build LightGBM from source with OpenCL flags; validate speedup & stability first.

Copilot, always honour this context: CPU-only XGBoost 2.x on Python 3.11 inside a Debian-slim Docker image, managed by Poetry, with monthly retrain and simple GitHub Actions CI/CD.

---
> Source: [BTG7/forecasting_gbdt](https://github.com/BTG7/forecasting_gbdt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-14 -->
