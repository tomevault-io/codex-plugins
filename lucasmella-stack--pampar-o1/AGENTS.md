# PAMPAr-o1 Copilot Instructions

## Definición en Una Frase

> **"PAMPAr-o1 es un cerebro artificial donde el tálamo orquesta tokens hacia territorios especializados (Expresivo, Contextual, Formal, Estructural) que colaboran via fronteras bidireccionales, combinando reglas explícitas (LLAVES 70%) con atención aprendida (30%) para generar lenguaje."**

## Project Overview
PAMPAr-o1 v9 is a **cerebral language model** with brain-inspired territorial architecture. The model uses 4 specialized "territories" containing 6 "neurons", coordinated by a central "thalamus" (Tálamo) that routes tokens using explicit rules (LLAVES) + learned attention.

## Architecture v9 (Territorial)

```
Input → Embedding → [BloqueTerrritorial ×N] → LM Head → Output
                          ↓
              Tálamo (LLAVES 70% + Atención 30%)
                          ↓
    ┌─────────────────────┴─────────────────────┐
    │                                           │
    ▼                                           ▼
┌───────────────┐                    ┌───────────────┐
│   EXPRESIVO   │◄───── Frontera ───►│  CONTEXTUAL   │
│ Lang + Creat  │                    │   Contexto    │
└───────┬───────┘                    └───────┬───────┘
        │                                    │
        │◄────── Fronteras Bidirec ─────────►│
        │                                    │
┌───────▼───────┐                    ┌───────▼───────┐
│    FORMAL     │◄───── Frontera ───►│ ESTRUCTURAL   │
│    Lógica     │                    │ Patrón + Mat  │
└───────────────┘                    └───────────────┘
                          ↓
                       Axiomas (opcional)
```

### Key Components in `pampar/cerebro/`
- **model.py** → Re-exports from `model_v9.py`
- **model_v9.py** → `PampaR` main class, `BloqueTerrritorial` processing blocks
- **talamo.py** → `TalamoTerritorial` orchestrator with LLAVES + territorial routing
- **territorio.py** → `Territorio`, `GestorTerritorios` - 4 functional groupings
- **frontera.py** → `FronteraBidireccional`, `GestorFronteras` - 6 bidirectional connections
- **modulos/especializados.py** → 6 specialized neurons (`NeuronaLenguaje`, `NeuronaLogica`, etc.)
- **razonamiento/axiomas.py** → `MotorAxiomas` with modus ponens, silogismo, etc.

### Territories (4)
| Territory | Modules | Function |
|-----------|---------|----------|
| **Expresivo** | Lenguaje + Creatividad | Generate fluent text, new ideas |
| **Contextual** | Contexto | Working memory, coherence |
| **Formal** | Lógica | Reasoning, rules |
| **Estructural** | Patrones + Matemáticas | Sequences, numbers, patterns |

### Frontiers (6 bidirectional)
All territories connect via bidirectional frontiers with learned gates.

### Configuration System
- **config.py** → `ConfigPampaR` for language model (v9)
- Presets: `LOCAL_4GB`, `LOCAL_4GB_MAX`, `SERVER_8GB`, `SERVER_24GB`, `SERVER_80GB`

## Developer Workflows

### Training
```bash
python scripts/train.py                     # Basic training
python scripts/train.py --tokens 10M        # Limit tokens (10M, 50M)
python scripts/train.py --resume            # Resume from checkpoint
python scripts/train.py --batch-size 32 --lr 1e-4 --epochs 10
```

### Testing v9
```bash
python scripts/test_v9.py                   # Test territorial architecture
pytest tests/                               # Run all tests
```

### Inference/Chat
```bash
python scripts/chat.py
python scripts/chat.py --checkpoint checkpoints/pampar_best.pt
```

### Data Requirements
- Tokenizer: `data/tokenizer/llarri_bpe.model` (SentencePiece BPE)
- Corpus: `data/wikitext-103/wikitext-103-raw/wiki.train.raw`
- Run `python scripts/download_corpus.py` to fetch WikiText-103

## Code Conventions

### File Headers
All Python files use SPDX license headers:
```python
# SPDX-License-Identifier: AGPL-3.0-or-later
# Copyright (c) 2024-2026 Lucas Ricardo Mella Chillemi
```

### Naming Conventions
- Spanish names for domain concepts: `Tálamo`, `Territorio`, `Frontera`, `Neurona`, `Axiomas`, `LLAVES`
- English for standard ML terms: `forward`, `batch`, `embedding`
- Configuration classes: `ConfigPampaR`
- Territories: `TipoTerritorio.EXPRESIVO`, `.CONTEXTUAL`, `.FORMAL`, `.ESTRUCTURAL`
- Module neurons: `NeuronaLenguaje`, `NeuronaLogica`, etc.

### Territory Pattern (v9)
Each territory in `territorio.py`:
1. Contains related modules sharing a buffer
2. Processes with `procesar()` or `procesar_basal()` (low activation)
3. Communicates with other territories via `frontera.py`

### Tálamo LLAVES System
The thalamus uses 70% rule-based (`peso_llaves=0.7`) + 30% learned attention:
```python
# In talamo.py - TalamoTerritorial
self.llaves = {
    'lenguaje': LlaveModulo(patrones=['el', 'la', 'de', 'que', ...]),
    'matematicas': LlaveModulo(patrones=['0-9', '+', '-', '=', ...]),
    # ...
}
# Modules aggregate to territories via modulo_a_territorio matrix
```

### Frontier Connections (v9)
Bidirectional connections between territories in `frontera.py`:
```python
FRONTERAS_DEFINIDAS = [
    ConfigFrontera('expresivo', 'contextual', 0.8),    # High: narrative + context
    ConfigFrontera('expresivo', 'formal', 0.5),        # Medium: argumentation
    ConfigFrontera('formal', 'estructural', 0.7),      # High: math logic
    # ...
]
```

## Key Implementation Details

### Checkpointing
- Checkpoints save: `{'model': state_dict, 'config': dict, 'epoch': int, 'val_loss': float}`
- Resume with: `torch.load(path, map_location=device, weights_only=False)`

### Mixed Precision
- Enabled via `config.use_mixed_precision = True`
- Uses `torch.amp.autocast` and `GradScaler`

### Gradient Checkpointing
- Enabled via `config.use_gradient_checkpointing = True`
- Applied in `CerebralBlock.forward()` using `torch.utils.checkpoint.checkpoint`

### Tokenizer Registration
```python
model.registrar_tokenizer(tokenizer)  # Populates LLAVES token mappings
```

## Important Notes
- **Bilingual docs**: English and Spanish versions exist (`*.es.md`)
- **Legacy code**: `versions/legacy/` contains older v4 MNIST implementation and v8 sinapsis
- **Diagrams**: Architecture diagrams in `diagrams/v9-territorial/`
- **License**: AGPL-3.0-or-later (copyleft, disclose source for derivatives)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/lucasmella-stack)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/lucasmella-stack)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
