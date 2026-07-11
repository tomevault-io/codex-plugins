# Copilot instructions for ERRA

Purpose: give AI coding agents the minimum context to be productive in this repo in minutes.

## Big picture
- Python package lives in `src/erra/` with the public API exposed via `erra.__init__`:
  - `erra()` (core analysis) returns `ERRAResult` (see `erra_core.py`)
  - `plot_erra_results()` (plots) in `utils.py`
- Example scripts:
  - `examples/*.py` use the installed package `erra` (requires `pip install -e .` or `PYTHONPATH=src`)
  - `code/python-version-example/*.py` is a self-contained demo that also ships a local `erra.py` for quick runs; prefer the package under `src/erra` for real work
- Reference datasets are not in `code/…`; they are under `reference_materials/R_implementation/demonstration-scripts/Source data/`.

## What to import and where it lives
- Primary entry point: `from erra import erra, ERRAResult, plot_erra_results`
  - Implementation: `src/erra/erra_core.py` (advanced ERRA, nonlinear, splitting, robust options)
  - Utilities & plotting: `src/erra/utils.py`
  - Nonlinear helpers: `src/erra/nonlin.py` (x’ transform, NRF labels)
  - Non-stationary splitting: `src/erra/splitting.py`

## Dev and run workflows (PowerShell)
- Create/activate an environment, then install in editable mode:
  ```powershell
  python -m pip install -U pip; pip install -e .[dev]
  ```
- Running examples that import the package (recommended):
  ```powershell
  # From repo root
  $env:PYTHONPATH = "$(Get-Location)\src"  # optional if not installed
  python examples\master_demonstration.py
  ```
- Data access for demos: set the base folder or rely on defaults
  ```powershell
  # Folder containing files like "MOPEX SacoR hourly.txt"
  $env:ERRA_DATA_DIR = "${PWD}\reference_materials\R_implementation\demonstration-scripts\Source data"
  ```
  The demo `code/python-version-example/example_en.py` auto-detects this path and also respects `ERRA_DATA_DIR`.

## Common pitfalls and fixes
- ModuleNotFoundError: `No module named 'erra'`
  - Cause: running examples without installing package
  - Fix: `pip install -e .` or set `PYTHONPATH=src` for the session
- FileNotFoundError for MOPEX data
  - Cause: scripts looked in `code/demonstration-scripts/...`
  - Current behavior: `example_en.py` now searches the repo and defaults to `reference_materials/R_implementation/demonstration-scripts/Source data`; or set `ERRA_DATA_DIR`.
- Type hints across two ERRAResult classes
  - The quick demo in `code/python-version-example/erra.py` defines its own `ERRAResult`, which differs from `src/erra/erra_core.ERRAResult` and may trigger static checker noise. Prefer the `src/erra` package for development code.

## API usage patterns (concrete)
- Linear analysis (hourly data, light smoothing):
  ```python
  from erra import erra, plot_erra_results
  result = erra(p=df[["p"]], q=df["q"], m=60, nu=0.1, dt=1/24, agg=1, labels=["P"]) 
  plot_erra_results(result, observed_q=df["q"], output_dir="figures")
  ```
- Nonlinear analysis with intensity knots (percentiles):
  ```python
  result = erra(p=df[["p"]], q=df["q"], m=60, xknots=[50,80,95], xknot_type="percentiles")
  ```
- Non-stationary splitting by a covariate:
  ```python
  split_params = {
    'crit': [df["q"].to_numpy()], 'crit_label': ['Wetness'], 'crit_lag': [1],
    'pct_breakpts': [True], 'breakpts': [[50, 90]], 'thresh': [0], 'by_bin': [True]
  }
  result = erra(p=df[["p"]], q=df["q"], m=60, split_params=split_params)
  ```

## Conventions and defaults
- Time unit set by `dt` (e.g., `6/24` for 6-hour steps). `m` is max lag in those units.
- `fq` applies a running-quantile detrend on discharge with window `4*m+1`.
- `nu` is Tikhonov smoothness; start at `0.01–0.1` for noisy data.
- Broken-stick (`nk`) compresses lags for speed; `nk≈10–20` is a good compromise.

## Where to look when changing behavior
- Build/packaging: `pyproject.toml` (package `erra-hydrology`, sources in `src`)
- Public exports: `src/erra/__init__.py`
- Algorithmic core and return shape: `src/erra/erra_core.py` (`ERRAResult` dataclass)
- Plot outputs and filenames: `src/erra/utils.py`
- Nonlinear transform and labels: `src/erra/nonlin.py`
- Covariate-based splitting: `src/erra/splitting.py`

## Quick verification
- Minimal smoke test from repo root (after install):
  ```powershell
  python - <<'PY'
  import pandas as pd, numpy as np
  from erra import erra
  p = pd.DataFrame({"p": np.random.rand(200)})
  q = np.random.rand(200)
  res = erra(p=p, q=q, m=10, nu=0.01)
  print(res.design_shape, len(res.lags))
  PY
  ```

If anything here is unclear or missing, ask for the specific workflow or file you need clarified and we’ll refine this guide.

## 中文说明（简版）
- 代码结构与入口
  - 核心包在 `src/erra/`；公共 API 通过 `erra.__init__` 暴露：`erra()`、`plot_erra_results()`、`ERRAResult`
  - 示例：`examples/*.py` 依赖已安装的包；`code/python-version-example/*.py` 为快速演示（内含本地 `erra.py`，开发时更推荐 `src/erra`）
  - 演示数据位于 `reference_materials/R_implementation/demonstration-scripts/Source data/`

- 常用导入
  - `from erra import erra, ERRAResult, plot_erra_results`
  - 实现位置：`erra_core.py`（算法核心/非线性/分割/鲁棒），`utils.py`（绘图），`nonlin.py`（x'、NRF），`splitting.py`（按协变量分割）

- 开发与运行（PowerShell）
  ```powershell
  # 可编辑安装
  python -m pip install -U pip; pip install -e .[dev]

  # 运行依赖已安装包的示例（亦可用临时 PYTHONPATH）
  $env:PYTHONPATH = "$(Get-Location)\src"  # 可选
  python examples\master_demonstration.py

  # 数据目录（包含 “MOPEX SacoR hourly.txt” 等）
  $env:ERRA_DATA_DIR = "${PWD}\reference_materials\R_implementation\demonstration-scripts\Source data"
  ```
  说明：`code/python-version-example/example_en.py` 会自动搜索上述目录，并支持 `ERRA_DATA_DIR` 环境变量。

- 常见问题
  - 无法导入 `erra`：未安装或未设置 `PYTHONPATH=src`；按上文安装或设置环境变量
  - 找不到数据文件：历史脚本指向 `code/demonstration-scripts/...`；现已默认搜索 `reference_materials/.../Source data`，或手动设置 `ERRA_DATA_DIR`
  - 类型提示冲突：演示版 `code/python-version-example/erra.py` 的 `ERRAResult` 与包内类型不同；开发时优先使用 `src/erra` 包

- 典型用法
  - 线性分析：
    ```python
    from erra import erra, plot_erra_results
    res = erra(p=df[["p"]], q=df["q"], m=60, nu=0.1, dt=1/24, agg=1, labels=["P"]) 
    plot_erra_results(res, observed_q=df["q"], output_dir="figures")
    ```
  - 非线性（强度节点）：
    ```python
    res = erra(p=df[["p"]], q=df["q"], m=60, xknots=[50,80,95], xknot_type="percentiles")
    ```
  - 非平稳（按协变量分割）：
    ```python
    split_params = {
      'crit': [df["q"].to_numpy()], 'crit_label': ['Wetness'], 'crit_lag': [1],
      'pct_breakpts': [True], 'breakpts': [[50, 90]], 'thresh': [0], 'by_bin': [True]
    }
    res = erra(p=df[["p"]], q=df["q"], m=60, split_params=split_params)
    ```

- 约定与默认
  - `dt` 定义时间单位（如 `6/24` 表示 6 小时）；`m` 为最大时滞（同单位）
  - `fq` 在 `4*m+1` 窗口上做运行分位去趋势
  - `nu` 为 Tikhonov 平滑；噪声数据建议从 `0.01–0.1` 试起
  - 断棍压缩 `nk≈10–20` 较折中

- 修改行为的入口
  - 打包与配置：`pyproject.toml`（包名 `erra-hydrology`，源码在 `src`）
  - 导出符号：`src/erra/__init__.py`
  - 算法核心/返回结构：`src/erra/erra_core.py`（`ERRAResult`）
  - 绘图与输出：`src/erra/utils.py`
  - 非线性与标签：`src/erra/nonlin.py`
  - 协变量分割：`src/erra/splitting.py`

- 快速校验（安装后）
  ```powershell
  python - <<'PY'
  import pandas as pd, numpy as np
  from erra import erra
  p = pd.DataFrame({"p": np.random.rand(200)})
  q = np.random.rand(200)
  res = erra(p=p, q=q, m=10, nu=0.01)
  print(res.design_shape, len(res.lags))
  PY
  ```

---
> Source: [licm13/ERRA](https://github.com/licm13/ERRA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-11 -->
