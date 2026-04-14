# LaMB Vertical - AI Coding Guidelines

## Architecture Overview

**LaMB (Latent Memory Bridge)** is a memory-augmented language model for RAG that compresses long contexts into fixed-size memory tokens using a vertical bridge mechanism.

### Core Components

- **`LaMBModel`** (`lamb/model.py`): Main model wrapping a base transformer with PEFT LoRA adapters and memory bridge
- **`VerticalLatentMemoryBridge`** (`lamb/bridge.py`): Compresses layer-wise hidden states into KV memory tokens for efficient attention
- **Training**: Uses KL divergence between teacher (full context) and student (memory-compressed) forward passes

### Key Patterns

**PEFT Integration**: Base models are typed as `Any` due to dynamic adapter methods. Use `cast(torch.nn.Module, model)` for device operations.

```python
self.base_model: Any  # PEFT-enhanced model with adapter methods
cast(torch.nn.Module, self.base_model).to(device=device)
```

**Configuration**: Uses dataclasses with factory functions for device/dtype detection. Environment variables override defaults.

```python
device: str = field(default_factory=pick_device)
dtype: torch.dtype = field(default_factory=lambda: pick_dtype(pick_device()))
```

**Text Processing**: Training splits chat-formatted text at assistant markers for context/target separation.

```python
splitter = "<|im_start|>assistant\n"
ctx_str, tgt_str = txt.split(splitter, 1)
ctx_str += splitter  # Include splitter in context
```

## Development Workflow

**Dependency Management**: Use `uv` exclusively. Never use `pip` directly.

```bash
uv sync                    # Install dependencies
uv sync --extra dev        # Add development tools
uv run lamb --help         # Run CLI commands
```

**Code Quality**: All code quality tasks handled by ruff (linting, formatting, import sorting).

```bash
uv run ruff check lamb/         # Lint with auto-fixable rules
uv run ruff check --fix lamb/   # Lint and auto-fix issues
uv run ruff format lamb/        # Format code
./scripts/improve_code.sh       # Auto-format, fix, and report errors
```

**Important**: After any code changes or edits, always run `./scripts/improve_code.sh` to ensure code quality and fix any linting issues before proceeding.

**Validation**: Use the utility scripts for comprehensive checks and fixes.

```bash
./scripts/report_error.sh   # Check syntax + linting, generate report
./scripts/format_code.sh    # Auto-format code with ruff
./scripts/improve_code.sh   # Run all fixes and report remaining issues
```

## Project Conventions

**Type Annotations**: Python 3.10+ compatible - use `X | Y` not `Union[X, Y]`, `list[int]` not `List[int]`.

**Error Handling**: Training gracefully handles non-finite losses by skipping batches, not crashing.

**Device Management**: Always use `pick_device()`, `pick_dtype()`, `pick_attn_implementation()` for cross-platform compatibility.

**Memory Tokens**: Configurable via `num_memory_tokens` (default: 32). Bridge compresses context into this fixed size.

**LoRA Configuration**: Applied to attention and MLP layers for compression. Only these parameters are trainable.

**Debugging**: Use `debug_reproduce_training()` for overfitting validation during training steps.</content>
<parameter name="filePath">/Users/anhvth/projects/ml-clara-v2/.github/copilot-instructions.md

NOTE:
Always run ./scripts/improve_code.sh to ensure code quality before returning to user

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/anhvth) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
