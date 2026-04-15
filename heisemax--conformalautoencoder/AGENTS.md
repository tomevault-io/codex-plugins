# Copilot Instructions for ConformalAutoencoder

This research codebase implements various autoencoder architectures for geometric learning experiments, focusing on conformal and isometric mappings.

## Architecture Overview

The project follows a hierarchical autoencoder class design in `Autoencoders.py`:

- **Base `Autoencoder`**: Standard reconstruction with MSE loss
- **`VariationalAutoencoder`**: Adds KL divergence regularization 
- **`IsometricAutoencoder`**: Preserves distances via `isometry_loss`
- **`ScaledIsometricAutoencoder`**: Uses `scaled_isometry_loss` variant
- **`ConformalAutoencoder`**: Core research focus - preserves angles via conformality constraints

All autoencoders share the same training interface but implement specialized `get_metrics()` and `get_loss()` methods for their respective objectives.

## Key Components

### Loss Functions (`metrics.py`)
- `conformality_trace_loss`: Measures angle preservation using Jacobian trace
- `isometry_loss`/`scaled_isometry_loss`: Distance preservation constraints
- `regularization`: Additional smoothness penalty
- `evaluate_conformality`: Comprehensive conformality assessment

### Data Generation (`data.py`)
- `make_half_sphere()`: Primary synthetic dataset for geometric experiments
- Generates noisy points on upper hemisphere with continuous labels

### Model Persistence
- Models saved to `models/` with lambda parameter encoding: `lambda_{value}_model_{run}.pth`
- Use built-in `save_checkpoint()`/`load_model_from_checkpoint()` methods
- Helper functions in `helper.py` for optimizer/scheduler state management

## Experimental Patterns

### Hyperparameter Naming Convention
- `lambda_conf`: Conformality loss weight
- `lambda_reg`: Regularization term weight  
- `lambda_iso`: Isometry loss weight
- `lambda_aug`: Data augmentation interpolation range
- Schedule variants: `lambda_conf_schedule`, `lambda_reg_schedule`

### Notebook Structure
Comparison notebooks follow this pattern:
1. **Setup**: Import modules, define encoder/decoder architectures
2. **Data**: Generate half-sphere dataset with train/val split
3. **Training Loop**: Multiple lambda values × multiple random seeds
4. **Evaluation**: Conformality metrics, reconstruction error, visualization
5. **Analysis**: DataFrame aggregation with statistical comparisons

Example notebook naming: `{method}_comp_halfsphere.ipynb` for systematic comparisons.

### Training Workflow
```python
# Standard training pattern
model = ConformalAutoencoder(encoder, decoder, lambda_conf=0.1, lambda_reg=0.01)
optimizer, scheduler = model.train_model(train_dataloader, val_dataloader, epochs=1000)
model.save_checkpoint(f"models/{experiment_name}/lambda_{lambda_val}_model_{run}.pth")
```

### Evaluation Pattern
```python
# Conformality assessment
stats = evaluate_conformality(model, val_data, double_precision=True)
# Returns dict with lambda_mean, lambda_std, off_diagonal metrics, etc.
```

## Development Guidelines

### Adding New Autoencoder Types
1. Inherit from `Autoencoder` base class
2. Override `get_metrics()` to return loss components as list
3. Override `get_loss()` to combine components with weights
4. Update `metrics_list` and `val_metrics_list` dictionaries
5. Implement specialized logging in `log_loss_and_metrics()`

### Geometric Loss Implementation
- Use `get_batch_jacobian()` for efficient Jacobian computation with chunking
- Employ `get_JTJ_trace_estimate()` for memory-efficient trace estimation
- Always include `torch.cuda.empty_cache()` after heavy tensor operations

### Experiment Organization
- Create comparison notebooks for systematic parameter sweeps
- Use consistent random seeds across experiments
- Save models with descriptive lambda-based naming
- Generate pandas DataFrames for statistical analysis
- Include both individual model results and aggregated statistics

The codebase prioritizes geometric interpretability over raw performance, focusing on understanding how different regularization terms affect the learned manifold structure.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/HeiseMax) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
