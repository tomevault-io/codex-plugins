# Copilot instructions for `vmaf-rs2`

## Build, test, and lint

These commands were validated in this workspace:

```bash
cargo build --workspace --quiet
cargo test --workspace --quiet
cargo fmt --all --check
cargo clippy --workspace --all-targets --quiet
```

Use these when you need a narrower scope:

```bash
cargo test -p vmaf
cargo test -p vmaf-model
cargo test -p vmaf-cli
```

To run a single test, prefer the package plus the fully-qualified inline test name:

```bash
cargo test -p vmaf 'tests::batch_and_sequential_paths_match' -- --exact
```

Tests are mostly inline `#[cfg(test)]` modules inside `src/*.rs`, so exact matching often needs the `tests::...` path rather than only the bare function name.

To exercise the CLI against bundled fixtures:

```bash
cargo run -p vmaf-cli -- \
  --reference videos/original.y4m \
  --distorted videos/noise.y4m \
  --model models/vmaf_v0.6.1.json
```

## High-level architecture

`docs/VMAF_DESIGN_SPEC.md` is the source of truth for algorithm behavior. The workspace is organized around that spec:

- `crates/vmaf-cli/src/main.rs` is the CLI entrypoint. It reads Y4M inputs, converts the luma plane to `u16`, loads a JSON model, and drives the scoring pipeline.
- `crates/vmaf/src/lib.rs` contains `VmafContext`, the orchestration layer. It owns the model plus the three feature extractors, buffers pending frames, finalizes per-frame scores, and pools the final score.
- `crates/vmaf-vif/` computes 4-scale integer VIF features.
- `crates/vmaf-adm/` computes integer ADM features.
- `crates/vmaf-motion/` computes Motion2 from reference frames only.
- `crates/vmaf-model/` parses model JSON/LIBSVM data, normalizes features, runs Nu-SVR inference, applies score transforms, and pools frame scores.
- `crates/vmaf-cpu/` provides runtime SIMD backend selection plus alignment/casting helpers shared by SIMD-capable crates.

The end-to-end flow is:

1. CLI reads reference/distorted Y4M frames.
2. `VmafContext` feeds each pair through VIF and ADM plus reference-only Motion.
3. Once all features for a frame are available, `vmaf` assembles them in model order and calls into `vmaf-model`.
4. `vmaf-model` normalizes features, runs SVM prediction, denormalizes, optionally applies `score_transform`, and returns a per-frame score.
5. `VmafContext` pools the finalized frame scores with the requested method and subsampling factor.

## Key conventions

- Preserve spec conformance. Crate docs and comments explicitly map code to sections in `docs/VMAF_DESIGN_SPEC.md`; when changing algorithmic code, keep the implementation aligned with the referenced section instead of “simplifying” math or reorderings.
- `VmafContext` is stateful, and motion has a lagged output. Always account for the final `flush()` call; otherwise the last pending Motion2-backed score is never emitted.
- The score feature order is fixed in `crates/vmaf/src/lib.rs`: `[adm2, motion2, vif_scale0, vif_scale1, vif_scale2, vif_scale3]`. Do not change that ordering unless the model pipeline changes with it.
- The CLI keeps `score_transform` disabled by default to match the reference `./vmaf` behavior for bundled `v0.6.x` models. Only enable it when behavior should follow the spec-mode transform path.
- Model parsing is stricter than a casual JSON reader. `crates/vmaf-model/src/json.rs` validates feature names and `feature_opts_dicts`, including consistent `vif_enhn_gain_limit` values across all VIF scales.
- SIMD is selected at runtime through `vmaf-cpu`. ADM, VIF, and Motion tests commonly compare scalar output against detected or explicit SIMD backends; keep that scalar-vs-SIMD parity intact when changing kernels.
- Tests prefer deterministic synthetic planes and bit-exact assertions (`to_bits()`), especially in VIF/ADM/model scoring paths. Follow that pattern instead of loose float tolerances unless the surrounding code already uses tolerances.
- Bundled production models live in `models/`, and several tests load them with `include_str!(...)`. Reuse those fixtures rather than inventing ad hoc model strings when possible.
- Existing `cargo clippy` runs exit successfully but emit warnings in SIMD-heavy code; do not assume every warning is new unless your change touched that area.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nusu-github)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/nusu-github)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
