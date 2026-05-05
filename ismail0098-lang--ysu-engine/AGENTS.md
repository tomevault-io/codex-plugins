# Copilot / AI agent instructions for YSU engine

This repo is a C-based raytracer, GPU compute engine, mesh editor, and physics simulator. The goal of these notes is to help an AI coding agent become productive quickly with minimal hand-holding.

## Repository layout

```
src/
  core/       — math (vec2-4, ray, camera), primitives (sphere, triangle), image, material, color
  render/     — CPU renderer, BVH, scene loader, G-buffer, postprocess, ysu_main.c entrypoint
  denoise/    — bilateral, neural, ONNX denoiser modules
  nerf/       — NeRF SIMD inference, hash grid, batch scheduler, depth hints
  vulkan/     — GPU Vulkan pipelines, LBVH builder, GPU BVH, OBJ loader
  physics/    — quantum volume raymarcher, nuclear fission/fusion, reactor thermal sim
  editor/     — mesh editor, viewport, edit mode, OBJ exporter (requires raylib)
  upscale/    — neural upscaling pipeline
  tools/      — standalone CLI utilities (inspect_ppm, ysub_info, ysub_to_ppm)
  third_party/— stb_image_write.h and other vendored headers
shaders/      — GLSL compute shaders + compiled .spv files
docs/         — all markdown documentation and notes
scripts/
  build/      — build scripts (.bat, .ps1)
  test/       — test and benchmark scripts
  analysis/   — Python analysis, training, and data scripts
output/
  renders/    — generated .ppm and .png images
  benchmarks/ — FPS/performance .csv results
  logs/       — debug and test output text files
build/        — compiled .o and .exe artifacts (gitignored)
DATA/         — scene files, .ysub binary data
models/       — pretrained NeRF model binaries
```

- **Big picture:** The primary executable is a CPU raytracer. Rendering is driven from `src/render/ysu_main.c` which prepares a `Camera` and calls the renderer in `src/render/render.c` (single-threaded `render_scene_st` or thread-pool `render_scene_mt`). Geometry/acceleration is implemented across `src/core/sphere.c`, `src/core/triangle.c`, and `src/render/bvh.c`. Postprocessing / denoising hooks live in `src/denoise/`.

- **Key files to read first:**
  - **Renderer & threading:** `src/render/render.c`
  - **CLI / entrypoint:** `src/render/ysu_main.c`
  - **Math & ray types:** `src/core/vec3.c`, `src/core/ray.c`, `src/core/camera.c`
  - **Primitives & BVH:** `src/core/sphere.c`, `src/core/triangle.c`, `src/render/bvh.c`
  - **Image / output:** `src/core/image.c`
  - **Editor / viewport:** `src/editor/ysu_mesh_edit.c`, `src/editor/ysu_viewport.c`
  - **GPU Vulkan pipeline:** `src/vulkan/gpu_vulkan_demo.c`
  - **Physics sims:** `src/physics/quantum_volume.c`, `src/physics/reactor_thermal.c`

- **Build system (CMake):**
  ```bash
  mkdir build && cd build
  cmake .. -DCMAKE_BUILD_TYPE=Release
  cmake --build .
  ```
  CMake auto-detects Vulkan, raylib, OpenMP, AVX2. Missing deps just skip those targets.

- **Runtime configuration (important):** The program is controlled exclusively via environment variables (see `ysu_main.c` and `render.c`). Useful variables:
  - `YSU_W`, `YSU_H` : image width / height
  - `YSU_SPP` : samples per pixel
  - `YSU_DEPTH` : max bounce depth
  - `YSU_THREADS` : thread count (0 = autodetect)
  - `YSU_TILE` : tile size for MT renderer
  - Adaptive sampling toggles in `render.c`: `YSU_ADAPTIVE`, `YSU_SPP_MIN`, `YSU_SPP_BATCH`, `YSU_REL_ERR`, `YSU_ABS_ERR`
  - `YSU_NEURAL_DENOISE` and `YSU_DUMP_RGB` control postprocessing and optional dumps

- **Include convention:** All `#include` directives use flat names (e.g., `#include "vec3.h"`). The CMake build adds `-I` flags for every `src/` subdirectory so headers resolve without path prefixes. **Do not** add subdirectory prefixes to includes.

- **Project-specific conventions & gotchas:**
  - Prefix convention: many modules and globals use `ysu_` (eg. `ysu_neural_denoise_maybe`, `ysu_rng_next01`). Follow that naming when adding helpers.
  - Deterministic RNG seeding is implemented per-tile / per-pixel in `render.c`. Changing seeding affects reproducibility tests.
  - The MT renderer uses a persistent thread pool and per-thread `WorkerLocal` structs aligned to 64 bytes to avoid false sharing — be conservative when changing that struct layout.
  - Adaptive sampling statistics are collected with `_Atomic` counters in `render.c`; preserve atomic semantics if refactoring concurrency.
  - The mesh editor (`ysu_mesh_edit.c`) is a standalone, single-file immediate-mode UI using `raylib`/`raymath` with its own fixed limits (`MAX_VERTS`, `MAX_TRIS`). Keep edits to that file consistent with those limits.

- **Integration & external deps:**
  - There are optional/experimental denoiser modules: `src/denoise/neural_denoise.c`, `src/denoise/onnx_denoise.c` — these may require external libraries (ONNX runtime or prebuilt shaders). If modifying, search these files first to see required headers.
  - `shaders/` contains compiled SPV files — watch for GPU/compute integration when touching denoising code.

- **Where to make common changes:**
  - Change default render config: `src/render/ysu_main.c`
  - Modify shading / material logic: `src/core/material.c` and `ray_color_internal` stub in `src/render/render.c`
  - Change topology/editing logic: `src/editor/ysu_mesh_edit.c` and `src/editor/ysu_mesh_topology.c`
  - Add new Vulkan compute passes: `src/vulkan/`
  - Add new physics simulations: `src/physics/`

- **Examples to copy/paste when running locally:**
  - Fast debug (low samples, single-thread):
    - Linux / WSL: `YSU_W=320 YSU_H=180 YSU_SPP=4 YSU_THREADS=0 ./build/bin/ysu`
  - Enable adaptive sampling and denoise:
    - `YSU_ADAPTIVE=1 YSU_NEURAL_DENOISE=1 ./build/bin/ysu`

- **If you change public APIs:** prefer to add `ysu_`-prefixed wrappers and keep global state changes minimal — many modules expect plain C-style structs (no allocation wrappers) and rely on plain arrays.

If any section is unclear or you want more examples (compile flags for MSVC/MinGW, or deeper notes on `neural_denoise.c` / ONNX integration), tell me which area to expand. I'll iterate on this file.

---
> Source: [ismail0098-lang/YSU-engine](https://github.com/ismail0098-lang/YSU-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-04 -->
