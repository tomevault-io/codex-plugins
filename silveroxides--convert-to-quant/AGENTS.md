# Learned Rounding Quantization - Development Workspace

## Project Overview

**Purpose**: Research and development workspace for neural network quantization methods targeting the **ComfyUI inference platform**.

This workspace develops and tests quantization algorithms that convert PyTorch model weights from full precision (FP16/FP32/BF16) to low-precision formats (FP8 or INT8) using **SVD-based learned rounding optimization**. The quantized models are designed to be compatible with ComfyUI's `quant_ops.py` system and support multiple AI model architectures (Flux, T5-XXL, Hunyuan Video, etc.).

**Development Context**:
- **Primary Goal**: Experimenting with quantization algorithms and optimization strategies
- **Target Platform**: ComfyUI inference runtime with `QuantizedTensor` support
- **Output Format**: Quantized safetensors with ComfyUI-compatible metadata (`.comfy_quant`)
- **Approach**: Simple script-based implementations for rapid prototyping and testing

### Core Architecture

1. **Main Converter** ([convert_to_quant.py](convert_to_quant.py)): Experimental quantization script implementing various optimization strategies
2. **Quantization Engine** (`LearnedRoundingConverter` class): Research implementation of SVD-based optimization algorithms
3. **Layout System** ([quant_ops.py](quant_ops.py)): ComfyUI-compatible pluggable quantization format handlers via `QuantizedTensor` wrapper
4. **Triton Kernels** ([kernels/int8_kernels.py](kernels/int8_kernels.py), [kernels/int8_matmul.py](kernels/int8_matmul.py)): GPU-accelerated quantization/dequantization operations
5. **ComfyUI Integration Reference** ([ComfyUI_Custom_Nodes_Agent/](../ComfyUI_Custom_Nodes_Agent/)): Reference submodule for ComfyUI development patterns

## Key Concepts

### Learned Rounding (Core Innovation)

Standard quantization uses round-to-nearest, which is locally optimal but globally suboptimal. This tool uses **SVD-based optimization** to minimize quantization error in the most important directions:

1. Compute truncated SVD: `W ≈ U_k @ S_k @ V_k^T`
2. Optimize quantized values to minimize: `||U_k^T @ (W_dequant - W_orig) @ V_k||`
3. This preserves weight matrix structure while finding optimal per-element rounding decisions

The gradient derivation for INT8 is **critical**: `∂L/∂Q = ∂L/∂dq * scale` (multiply by scale, not divide) because dequantization multiplies `Q * scale`.

### Quantization Formats

- **FP8** (`float8_e4m3fn`): 8-bit floating point, range [-448, 448]
  - Scaling: per-tensor or per-block (partitions input dimension)
  - Best for modern GPUs with FP8 tensor cores
  
- **INT8** (block-wise): 8-bit integer, symmetric range [-127, 127]
  - **2D block-wise scaling** for weights: `(M//block_size, N//block_size)`
  - 1D block-wise scaling for activations: `(*batch_dims, K//block_size)`
  - Requires dimensions divisible by `block_size` (default 64/128)

### Optimizer Choices

Controlled via `--optimizer` flag:
- **`original`**: Custom adaptive learning rate, fastest convergence, no autograd overhead
- **`adamw`/`radam`**: PyTorch optimizers, good for general cases

Use `--simple` to skip learned rounding entirely (simple round-to-nearest).

## Code Patterns

### 1. Layout-Based Quantization (quant_ops.py)

The codebase uses a **pluggable layout system** to support multiple quantization formats without code duplication:

```python
# Register a new layout-specific operation
@register_layout_op(torch.ops.aten.linear.default, BlockWiseINT8Layout)
def int8_linear(func, args, kwargs):
    # Extract quantized data using get_plain_tensors()
    qdata, scale, block_size, is_weight = BlockWiseINT8Layout.get_plain_tensors(weight)
    # Perform quantized computation
    result = _int8_gemm_triton_or_fallback(...)
    return result
```

**Pattern**: Each layout (FP8, INT8) has a `QuantizedLayout` subclass that implements `quantize()`, `dequantize()`, and `get_plain_tensors()`. Operations are dispatched via `__torch_dispatch__`.

### 2. Model-Specific Layer Exclusions

The tool preserves sensitive layers (norms, biases, embeddings) in high precision using exclusion lists:

```python
# Example: T5-XXL exclusions
T5XXL_REMOVE_KEY_NAMES = ["decoder", "lm_head"]
AVOID_KEY_NAMES = ["norm", "bias", "embed_tokens", ...]
```

**Pattern**: Check layer names against exclusion lists before quantization. Use model-specific flags (`--t5xxl`, `--distillation_large`, etc.) to activate exclusions.

### 3. Bias Correction via Calibration Data

After quantization, biases are adjusted to compensate for quantization error:

```python
# Generate calibration data (once per input dimension)
calibration_data_cache[in_features] = torch.randn(calib_samples, in_features, ...)

# Correct bias
weight_error = W_orig - W_dequant
output_error = X_calib @ weight_error.T
bias_correction = output_error.mean(dim=0)
b_new = b_orig - bias_correction
```

**Pattern**: Use synthetic random data (no real dataset needed) to estimate output error distribution.

### 4. Triton Kernel Integration

INT8 quantization uses Triton kernels for performance:

```python
if _HAS_TRITON_INT8 and tensor.is_cuda:
    try:
        qdata, scale = triton_weight_quant(tensor, block_size=block_size)
    except Exception as e:
        # Fallback to PyTorch implementation
        qdata, scale = cls._weight_quantize_pytorch(tensor, block_size)
```

**Pattern**: Always provide PyTorch fallback for CPU or when Triton unavailable. Use `try/except` to gracefully degrade.
ComfyUI Integration

### Quantized Model Requirements

Quantized models must be compatible with ComfyUI's quantization system:

- **Metadata**: `.comfy_quant` tensor with JSON-encoded format information
- **Scale Storage**: `weight_scale` and optional `input_scale` tensors per layer
- **Layout Compatibility**: Matches ComfyUI's `QuantizedLayout` interface (see [quantization.examples.md](../quantization.examples.md))
- **Tensor Operations**: Compatible with `__torch_dispatch__` mechanism in ComfyUI's `quant_ops.py`

### Testing in ComfyUI

1. Load quantized model in ComfyUI
2. Verify `QuantizedTensor` wrappers are created correctly
3. Check inference quality (visual comparison, metrics)
4. Monitor memory usage and inference speed
5. Test with various samplers and schedulers

Reference: [ComfyUI_Custom_Nodes_Agent](../ComfyUI_Custom_Nodes_Agent/) for integration patterns

## Development Guidelines

### Experimenting with Quantization Algorithms

1. Modify `LearnedRoundingConverter._optimize_*()` methods to test new optimizers
2. Adjust SVD rank selection logic (`top_p`, `min_k`, `max_k`)
3. Experiment with different loss functions in optimization loop
4. Test scale calculation strategies (per-tensor vs per-block)
5. Document findings in `DEVELOPMENT.md`

### Adding Support for New Models

1. Identify sensitive layers (norms, embeddings, distillation layers) via model inspection
2. Add exclusion keynames to global lists (e.g., `MODEL_AVOID_KEY_NAMES`)
3. Add CLI flag (e.g., `--model_name`) and exclusion logic in `convert_to_fp8_scaled()`
4. Test quantized model in ComfyUI with representative prompts
5. Add CLI flag (e.g., `--model_name`) and exclusion logic in `convert_to_fp8_scaled()`
4. Document in [MANUAL.md](MANUAL.md) with example command

### Implementing New Quantization Formats

1. Create `QuantizedLayout` subclass in [quant_ops.py](quant_ops.py)
2. Implement `quantize()`, `dequantize()`, `get_plain_tensors()` methods
3. Register layout-specific handlers with `@register_layout_op` decorator
4. Add format to `QUANT_ALGOS` and `LAYOUTS` dicts
5. Update `LearnedRoundingConverter` to support new format

### Performance Optimization

- **Heuristics**: Use `--heur` to skip layers with poor quantization characteristics (see `should_skip_layer_for_performance()`)
- **Block sizes**: Larger blocks (128) reduce overhead but may increase quantization error
- **Triton kernels**: For INT8, `triton_v2` backend ([int8_matmul.py](kernels/int8_matmul.py)) has autotuning but is experimental

### Testing Considerations

- **Dimension validation**: INT8 requires dimensions divisible by `block_size`
- **Scale shape**: Weights use 2D scales `(M//bs, N//bs)`, activations use 1D `(*batch, K//bs)`
- **Zero tensors**: Skip optimization for all-zero tensors (see early return in `convert()`)
- **Device compatibility**: FP8 requires PyTorch 2.1+ and FP8-capable GPU (Ada Lovelace/Hopper)

## Common Workflows

### Run Quantization
```bash
python convert_to_quant.py -i model.safetensors --comfy_quant [--int8] [--t5xxl]
```

### Debug Optimization
- Add `print()` statements in `LearnedRoundingConverter._optimize_*()` methods
- Check loss convergence in progress bar (`best_loss` should decrease)
- Use `--simple` to isolate issues from learned rounding

### Profile Triton Kernels
- Set environment variable: `TRITON_PRINT_AUTOTUNING=1`
- Compare `--kernel_backend triton` vs `triton_v2` performance
- Fallback to PyTorch if Triton fails (logs warning)

## File Structure

```
convert_to_quant.py       # Experimental quantization implementations
quant_ops.py              # Layout system matching ComfyUI's QuantizedTensor interface
kernels/
  int8_kernels.py         # Triton kernels (default backend)
  int8_matmul.py          # Triton v2 kernels (autotuned, experimental)
float_utils/
  float.py                # Stochastic rounding utilities
MANUAL.md                 # User documentation for converted models
DEVELOPMENT.md            # Research notes and experimental findings
quantization.examples.md  # ComfyUI integration patterns and examples
ComfyUI_Custom_Nodes_Agent/ # Reference submodule (read-only) for ComfyUI patterns
README.md                 # Project overview
```

## Gotchas

1. **INT8 transpose**: When transposing INT8 weights, must also transpose scale tensor (see `int8_transpose()`)
2. **Gradient direction**: For INT8 learned rounding, gradient is **multiplied** by scale (chain rule from `dq = Q * scale`)
3. **ComfyUI metadata**: `.comfy_quant` tensor must be created with `dict_to_tensor()` for proper JSON encoding
4. **Weight format**: Triton kernels expect weights in `(N, K)` format (standard PyTorch), not `(K, N)`
5. **Scale minimum**: Always clamp scale to `1e-8` to avoid division by zero

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/silveroxides)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/silveroxides)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
