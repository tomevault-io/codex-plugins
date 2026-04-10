# Copilot Instructions for llcode (localcode)

## Overview

`llcode` (binary: `localcode`) is a Rust CLI tool for hardware-aware local LLM deployment. It detects available hardware (CPU/GPU/VRAM), recommends optimal models and quantizations, and manages inference server configuration (primarily via llama-swap on macOS).

## Build & Development Commands

- Debug build: `cargo build`
- Release build: `cargo build --release`
- Run CLI: `cargo run -- [COMMAND]` (commands: `init`, `list`, `download`, `start`, `stop`, `info`, `doctor`)
- Run all tests: `cargo test`
- Run tests matching pattern: `cargo test [pattern]` (e.g., `cargo test fit::`)
- Show stdout during tests: `cargo test -- --nocapture`
- Lint: `cargo clippy`
- Format: `cargo fmt`

## Architecture

- Single binary with core library in `src/core/`
- **main.rs**: CLI entry point using `clap`. Handles config generation and interactive model selection.
- **core/hardware.rs**: Hardware detection (CPU/GPU/VRAM, backend type, unified memory, multi-GPU aggregation)
- **core/models.rs**: Model database, quantization metadata, quantization hierarchy
- **core/fit.rs**: Fitting engine, determines FitLevel and RunMode, scoring based on memory, speed, quality
- **core/plan.rs**: Execution plans, hardware upgrade recommendations, throughput estimation
- **core/providers.rs**: ModelProvider trait, LlamaCppProvider (primary), MlxProvider (delegates to `hf` CLI), download/verify models, progress streaming

## Key Configuration Files

| File                     | Purpose                                                        |
|--------------------------|----------------------------------------------------------------|
| `localcode.json`         | User config: models dir, port, selected models, hardware, etc.  |
| `llama-swap-config.yaml` | llama-swap server config: model variants, command lines, hooks  |
| `opencode.json`          | OpenCode AI provider integration config                         |
| `src/core/data/hf_models.json` | HuggingFace model registry for recommendations         |

## Runtime & Platform Notes

- Primary platform: macOS (llama-swap via Homebrew)
- Model files (`*.gguf`) are gitignored, stored in `models/`
- InferenceRuntime enum: `LlamaCpp`, `MLX`, `Vllm`
- Hardware backend enum: Metal (Apple), CUDA (NVIDIA), ROCm (AMD), Vulkan, CPU (ARM/x86), Ascend NPU

## Conventions & Pitfalls

- Quantization hierarchy: Q8_0 > Q6_K > Q5_K_M > Q4_K_M > Q2_K, plus MLX/AWQ/GPTQ variants
- `score_fit` requires ≥15% memory headroom for `Good` fit; below that is `Marginal`
- Most tests are in `core/fit.rs`
- Model downloads use background threads, HTTP Range resume, and SHA-256 verification
- Progress is streamed via PullHandle/PullEvent and rendered with `indicatif`

## Example Prompts

- "Build and test the project"
- "List all supported quantization levels"
- "Show how hardware detection works"
- "Explain how model fit scoring is determined"
- "Where is the model download logic implemented?"

## Next Steps

- Consider agent customizations for: test coverage reporting, config file validation, or hardware compatibility checks.
- Use `applyTo` patterns for future instructions if splitting frontend/backend or test logic.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thewulf7)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/thewulf7)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
