# AI Coding Assistant Instructions

> These instructions apply to GitHub Copilot, Claude Code, and other AI coding assistants.

## Project Overview

This is a **GEO Survival Analysis** application - a full-stack bioinformatics web app that analyzes gene expression data from the NCBI Gene Expression Omnibus (GEO) database to identify genes associated with survival outcomes. The project uses AI/LLM for intelligent dataset ranking and analysis.

## Technology Stack

### Backend (Python 3.13+)
- **Framework:** FastAPI with Uvicorn
- **Data Processing:** Pandas, NumPy, SciPy
- **Survival Analysis:** lifelines (Kaplan-Meier, Cox regression)
- **LLM Integration:** Pydantic-AI (Mistral, Anthropic Claude)
- **Validation:** Pydantic 2.0+
- **HTTP Client:** httpx (async)
- **Package Manager:** uv

### Frontend (TypeScript)
- **Framework:** React 18 with TypeScript
- **State Management:** Redux Toolkit
- **Build Tool:** Vite
- **Styling:** Tailwind CSS
- **Visualization:** Recharts
- **HTTP Client:** Axios

## Code Style Guidelines

### Exception Handling
- **Never** create bare `try/except` blocks
- Always catch **specific exceptions** or omit try/except entirely
- Only use exception handling where at least one particular exception type can be defined

```python
# BAD - Never do this
try:
    result = process_data()
except:
    pass

# BAD - Too broad
try:
    result = process_data()
except Exception as e:
    logger.error(e)

# GOOD - Specific exceptions
try:
    result = process_data()
except (ValueError, KeyError) as e:
    logger.error(f"Data processing failed: {e}")
    raise

# GOOD - If no specific exception applies, don't use try/except
result = process_data()  # Let it fail naturally
```

### Environment & Commands
- Use `uv` to manage Python environments and run commands
- Activate environment: `uv sync` then use `uv run` prefix
- Example: `uv run python -m uvicorn app.main:app --reload`

### File Creation Policy
- **Never** create standalone explanation/documentation `.md` files
- Keep documentation in code comments or existing README files
- Prefer editing existing files over creating new ones

### Python Conventions
- Use `snake_case` for functions, variables, and modules
- Use `CamelCase` for class names
- Use type hints for all function signatures
- Use `async/await` for I/O operations
- Use dataclasses or Pydantic models for data structures
- Follow the service layer pattern (routes → services → clients)

```python
# Example service function signature
async def analyze_survival(
    dataset_id: str,
    gene_symbol: str,
    time_column: str,
    event_column: str,
) -> GeneSurvivalResult:
    """Perform survival analysis for a gene in a dataset."""
    ...
```

### TypeScript/React Conventions
- Use `camelCase` for variables and functions
- Use `PascalCase` for components and interfaces
- Use functional components with hooks
- Keep components focused and composable
- Use Redux for global state, local state for UI-only concerns

```typescript
// Example component structure
interface GeneCardProps {
  gene: GeneSurvivalResponse;
  isExpanded: boolean;
  onToggle: () => void;
}

const GeneCard: React.FC<GeneCardProps> = ({ gene, isExpanded, onToggle }) => {
  // Component implementation
};
```

## Project Structure

```
backend/
├── app/
│   ├── main.py              # FastAPI entry point
│   ├── api/routes.py        # API endpoints
│   ├── models/              # Pydantic models
│   │   ├── request_models.py
│   │   ├── response_models.py
│   │   └── llm_models.py
│   └── services/            # Business logic
│       ├── geo_client.py              # GEO API client
│       ├── geo_loader_service.py      # Dataset loading
│       ├── survival_analysis_service.py
│       └── geo_survival_workflow_orchestrator.py

frontend/
├── src/
│   ├── components/          # React components
│   ├── store/               # Redux state
│   │   ├── store.ts
│   │   └── searchSlice.ts
│   └── services/api.ts      # API client
```

## Key Services & Patterns

### Backend Services
- **GEOClient:** Handles NCBI GEO API requests with rate limiting
- **GEODataLoaderService:** AI-powered format detection for expression data
- **SurvivalAnalysisService:** Kaplan-Meier and Cox regression analysis
- **GEODatasetRankingService:** LLM-based dataset quality ranking

### Common Patterns
```python
# Service instantiation pattern
class SurvivalAnalysisService:
    def __init__(self, config: Optional[AnalysisConfig] = None):
        self.config = config or AnalysisConfig()
        self.logger = logging.getLogger(__name__)

    async def analyze(self, data: pd.DataFrame) -> AnalysisResult:
        ...

# Route handler pattern
@router.post("/search", response_model=AnalysisResponse)
async def search_datasets(request: AnalysisRequest) -> AnalysisResponse:
    orchestrator = GEOSurvivalWorkflowOrchestrator()
    return await orchestrator.execute(request)
```

## API Reference

### Endpoints
| Method | Endpoint | Description |
|--------|----------|-------------|
| GET | `/api/health` | Health check |
| GET | `/api/models` | List available LLM models |
| POST | `/api/search` | Search and analyze datasets |

### Request/Response Models
```python
# Request
class AnalysisRequest(BaseModel):
    query: str
    max_datasets: int = 10
    organism: Optional[str] = None
    min_occurrence: int = 2
    model: str = "mistral"
    ranking_multiplier: int = 3

# Response includes survival genes with hazard ratios, p-values, KM curves
```

## Development Commands

### Backend
```bash
# Install dependencies
uv sync

# Run development server
uv run python -m uvicorn app.main:app --reload --host 0.0.0.0 --port 8000

# Run with production settings
uv run python -m uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4
```

### Frontend
```bash
# Install dependencies
npm install

# Development server (proxies to backend on :8000)
npm run dev

# Production build
npm run build

# Lint
npm run lint
```

## Domain Knowledge

### Survival Analysis Concepts
- **Kaplan-Meier:** Non-parametric survival curve estimation
- **Cox Proportional Hazards:** Semi-parametric regression for hazard ratios
- **Hazard Ratio (HR):** >1 indicates increased risk, <1 indicates protective
- **Log-rank test:** Compares survival distributions between groups
- **Gene Expression Omnibus (GEO):** NCBI's repository for expression data

### Data Flow
1. User query → LLM ranks relevant GEO datasets
2. Expression matrices downloaded and parsed
3. Probe IDs mapped to gene symbols
4. Survival metadata detected in clinical data
5. Cox regression identifies survival-associated genes
6. Results aggregated across datasets for consistency

## Testing Guidelines

When writing tests:
- Use pytest for backend testing
- Mock external API calls (GEO, LLM providers)
- Test service methods independently
- Use fixtures for common data structures

```python
# Example test structure
@pytest.fixture
def sample_expression_data():
    return pd.DataFrame({...})

async def test_survival_analysis(sample_expression_data):
    service = SurvivalAnalysisService()
    result = await service.analyze(sample_expression_data)
    assert result.p_value < 0.05
```

## Common Tasks

### Adding a New API Endpoint
1. Define request/response models in `backend/app/models/`
2. Implement service logic in `backend/app/services/`
3. Add route handler in `backend/app/api/routes.py`
4. Update frontend API client in `frontend/src/services/api.ts`

### Adding a New React Component
1. Create component file in `frontend/src/components/`
2. Use TypeScript interfaces for props
3. Connect to Redux store if needed
4. Add to parent component

### Implementing New Survival Analysis Methods
1. Add method to `SurvivalAnalysisService`
2. Use lifelines library for statistical computations
3. Return results in standardized response format
4. Update response models if adding new fields

## Error Handling Patterns

```python
# Backend - Use FastAPI's HTTPException
from fastapi import HTTPException

async def get_dataset(dataset_id: str) -> GEODataset:
    dataset = await geo_client.fetch(dataset_id)
    if not dataset:
        raise HTTPException(status_code=404, detail=f"Dataset {dataset_id} not found")
    return dataset

# Frontend - Handle in Redux slice
export const searchGenes = createAsyncThunk(
  'search/searchGenes',
  async (query: string, { rejectWithValue }) => {
    try {
      const response = await api.search(query);
      return response.data;
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);
```

## Logging

Backend uses centralized logging configuration:
```python
import logging
logger = logging.getLogger(__name__)

# Use appropriate log levels
logger.debug("Detailed info for debugging")
logger.info("General operational messages")
logger.warning("Something unexpected but not critical")
logger.error("Error that needs attention")
```

Logs are stored in `backend/geo_logs/` with rotation (50MB max).

## Environment Variables

Required:
- `MISTRAL_KEY` - Mistral API key for LLM features
- `EMAIL` - For NCBI GEO API requests (increases rate limit)

## Performance Considerations

- Use `async/await` for all I/O operations
- Cache platform mappings to avoid repeated downloads
- Batch gene analyses where possible
- Use parallel execution for independent dataset processing
- Frontend: Memoize expensive computations with `useMemo`/`useCallback`

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/svarogjk) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
