# AI Coding Agent Instructions for HT-HGNN v2.0 Supply Chain Risk Analysis

## Project Overview
Supply Chain Risk Analysis using **Heterogeneous Temporal Hypergraph Neural Networks (HT-HGNN v2.0)** - a production-ready ML platform that models complex multi-party supply chain interactions using hypergraph neural networks with temporal dynamics, heterogeneous relationship processing, cascading disruption simulation, and explainability.

**Key Innovation**: Combines spectral hypergraph convolution (multi-way relationships), Bi-LSTM + Transformer temporal fusion, heterogeneous relation attention (5 relation types), cascade risk scoring, and HyperSHAP explainability.

**v2.0 Enhancements over v1.0**: Multi-dataset support (5 datasets), dynamic hyperedge construction, cascade disruption simulation engine, HyperSHAP explainability, Transformer-augmented temporal encoding, new `co_disrupted_with` relation type.

---

## Critical Architecture: The Data-Model-API Pipeline

### 1. **Data Layer** (`src/data/`)
- **`DataAdapter`** (`data_adapter.py`): Unified normalization layer for all 5 datasets
- **Dataset-specific loaders**:
  - `DataCoLoader` (`dataco_loader.py`): DataCo logistics (180K records)
  - `BOMLoader` (`bom_loader.py`): Automotive BOM (12K components)
  - `PortDisruptionLoader` (`port_loader.py`): Global port disruptions (847 ports)
  - `MaintenanceLoader` (`maintenance_loader.py`): Predictive maintenance (10K records)
  - `RetailLoader` (`retail_loader.py`): M5 retail demand-supply (30K products)
- **Legacy**: `SupplyChainDataGenerator` (synthetic), `RealDataLoader` (generic CSV)
- Output: Standardized dict with `node_features`, `incidence_matrix`, `timestamps`, `node_types`, `edge_types`, `hyperedge_weights`

### 2. **Hypergraph Layer** (`src/hypergraph/`)
- **`Hypergraph`** (`hypergraph.py`): Core data structure with V, E, incidence, echelon dependencies
- **`DynamicHyperedgeConstructor`** (`dynamic_constructor.py`): Temporal co-occurrence mining for discovering emergent hyperedges
- **`RiskLabelGenerator`** (`risk_labels.py`): HCI formula with joint failure, engineering impact, propagation risk
- **`HypergraphSerializer`** (`visualization_utils.py`): D3/Cytoscape/vis-network graph serialization

### 3. **Model Layer** (`src/models/`)
- **v1.0 architecture** (`ht_hgnn_model.py`): HypergraphConvolution + HGT + TGN + 3 output heads
- **v2.0 spectral conv** (`hypergraph_conv.py`): `SpectralHypergraphConv` - Zhou et al. spectral formulation with residual + LayerNorm
- **v2.0 temporal** (`temporal_encoder.py`): `TemporalFusionEncoder` - Bi-LSTM + Transformer with learned gating
- **v2.0 relations** (`relation_fusion.py`): `HeterogeneousRelationFusion` - 5 relation types with softmax attention
- **v2.0 cascade** (`cascade_model.py`): `CascadeRiskHead` + `DynamicHyperedgeWeightLearner`
- **Baselines** (`baseline_models.py`): XGBoost, Random Forest, Gradient Boosting

### 4. **Explainability Layer** (`src/explainability/`)
- **`HyperSHAP`** (`hypershap.py`): Custom SHAP for hypergraphs - node, hyperedge, feature attribution
- **`HyperedgeImportanceAnalyzer`** (`hyperedge_importance.py`): Gradient, removal, attention methods
- **`FeatureAttributionAnalyzer`** (`feature_attribution.py`): Integrated gradients

### 5. **Simulation Layer** (`src/simulation/`)
- **`CascadeEngine`** (`cascade_engine.py`): Hypergraph Independent Cascade model
- **`StressTester`** (`stress_tester.py`): Bulk scenario testing (random, targeted, geographic)
- **`ScenarioBuilder`** (`scenario_builder.py`): What-if scenario construction

### 6. **Task Queue** (`src/tasks/`)
- **`celery_app.py`**: Celery + Redis configuration
- **`simulation_tasks.py`**: Async tasks for cascade simulation, stress testing, explanations

### 7. **Evaluation** (`src/evaluation/`)
- **`validation.py`**: AblationTester, FailureSimulator (v1.0)
- **`hypergraph_metrics.py`**: Hyperedge-aware accuracy, group risk consistency, NDCG
- **`cascade_eval.py`**: Cascade depth MAE, spread accuracy, timing accuracy

### 8. **API Layer** (`frontend/backend_server.py` - FastAPI)
- **v1.0 endpoints**: `/health`, `/model/info`, `/predict`, `/analyze`, `/upload/predict`, `/training/history`
- **v2.0 endpoints**: `/datasets`, `/datasets/{id}/load`, `/explain`, `/simulate/cascade`, `/tasks/{task_id}`, WS `/training/stream`
- 14 total endpoints, WebSocket support for live training

### 9. **Frontend** (`frontend/src/` - React 18 + TypeScript)
- **Pages**: DashboardPage, VisualizationPage, SimulationPage, ExplainPage
- **Components**: DatasetSelector, HypergraphCanvas (D3), CascadeSimulator, ExplainabilityPanel, RiskDashboard, ModelStats, FileUpload, Header
- **Hooks**: useHypergraph, useCascade, useExplain
- **Services**: inferenceService, datasetService, simulationService, explainService

---

## Essential Workflows

### Training a New Model (v2.0)
```bash
# Single dataset
python train_ht_hgnn.py --dataset dataco --epochs 100 --lr 0.001

# Transfer learning
python train_ht_hgnn.py --dataset all --pretrain bom --finetune dataco

# Resume from checkpoint
python train_ht_hgnn.py --dataset dataco --resume outputs/checkpoints/best_dataco.pt
```

### Running the Full System
```bash
# Option A: Docker
docker-compose up --build

# Option B: Manual
python frontend/backend_server.py          # Backend (port 8000)
cd frontend && npm install && npm run dev  # Frontend (port 5173)
celery -A src.tasks.celery_app worker      # Celery worker (for async tasks)
redis-server                               # Redis (for caching)
```

### Running Cascade Simulation
```bash
python scripts/simulate_cascade.py --dataset bom --shock-node NODE_001 --shock-magnitude 1.0 --time-steps 12
```

### Generating Explanations
```bash
python scripts/explain.py --dataset dataco --node-ids N001 N002 N150
```

---

## Multi-Task Learning (v2.0)
- 4 output heads: price (MSE), change (BCE), criticality (CE 4-class), cascade (KL divergence)
- Loss weights: `1.0*price + 0.8*change + 1.2*criticality + 0.6*cascade`
- 5 relation types: supplier_of, manufactured_by, transported_by, quality_controlled_by, co_disrupted_with

---

## Path Conventions
- Python source: `src/` (data, models, hypergraph, evaluation, explainability, simulation, tasks)
- CLI scripts: `scripts/` (download, build, simulate, stress test, explain)
- Training scripts: root level (`train_ht_hgnn.py`)
- Outputs: `outputs/{checkpoints, datasets, models, explanations, simulations, training_history}`
- Datasets: `Data set/{DataCo, BOM, Ports, Maintenance, Retail, processed}`
- Frontend: `frontend/src/{components, pages, hooks, services, types}`
- Tests: `tests/`
- Docker: root level (`Dockerfile.backend`, `Dockerfile.frontend`, `docker-compose.yml`)

---

## Key Files for Common Tasks

| Task | Primary File(s) |
|------|-----------------|
| Add new dataset | Create loader in `src/data/`, register in `DataAdapter` |
| Modify model | `src/models/hypergraph_conv.py`, `temporal_encoder.py`, `relation_fusion.py` |
| Train model | `train_ht_hgnn.py` (v2.0 with argparse) |
| Run inference | `frontend/backend_server.py` → `/predict` or `/upload/predict` |
| Cascade simulation | `src/simulation/cascade_engine.py` or `scripts/simulate_cascade.py` |
| Explainability | `src/explainability/hypershap.py` or `scripts/explain.py` |
| Hypergraph ops | `src/hypergraph/hypergraph.py`, `dynamic_constructor.py` |
| Risk computation | `src/hypergraph/risk_labels.py` |
| Frontend dashboard | `frontend/src/pages/DashboardPage.tsx`, `App.tsx` |
| Frontend simulation | `frontend/src/pages/SimulationPage.tsx`, `components/CascadeSimulator.tsx` |
| Frontend explain | `frontend/src/pages/ExplainPage.tsx`, `components/ExplainabilityPanel.tsx` |
| Stress testing | `src/simulation/stress_tester.py` or `scripts/stress_tester.py` |
| API endpoints | `frontend/backend_server.py` |
| Tests | `tests/test_*.py` (pytest) |

---

## Common Pitfalls to Avoid

1. **Incidence matrix dimensions**: H is (N_hyperedges x N_nodes) in hypergraph.py but spectral conv expects (N_nodes x N_hyperedges)
2. **Missing data normalization**: Always use DataAdapter to normalize features to [0,1]
3. **Dynamic vs static hyperedges**: DynamicHyperedgeConstructor adds edges at runtime; ensure incidence matrix is updated
4. **Cascade threshold sensitivity**: Default theta=0.5; lower values cause faster propagation
5. **HyperSHAP computation cost**: Exponential in hyperedge count; use max_hyperedges parameter to limit
6. **WebSocket connection**: Training stream requires active WS connection; handle reconnection
7. **Celery worker required**: Cascade simulation is async; celery worker must be running
8. **Dataset file sizes**: DataCo is 180MB+, M5 is multi-GB; use Polars for large datasets

---

## Development Notes
- **Python version**: 3.10+
- **PyTorch backend**: 2.1+ with CUDA 12.1 support
- **Frontend build**: Vite, React 18, TypeScript 5.2, TailwindCSS
- **Graph visualization**: D3.js v7 (HypergraphCanvas component)
- **Async tasks**: Celery 5.3 + Redis 7
- **Docker**: docker-compose for full stack deployment
- **Testing**: pytest with 40+ test cases in tests/

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Anshu007-ai)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Anshu007-ai)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
