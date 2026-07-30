# B3 Data Platform — Copilot Instructions

## Contexto do Projeto

Plataforma de dados financeiros (Data Lakehouse) para ações da B3,
implementando Arquitetura Medallion (Bronze → Silver → Gold → Report).

## Stack Tecnológica

| Categoria     | Tecnologias                                                       |
|---------------|-------------------------------------------------------------------|
| Processamento | Polars >= 0.20, PySpark 3.5.1, Delta Lake 3.1.0, PyArrow 14      |
| Ingestão      | yfinance >= 0.2.36, requests >= 2.31                              |
| Armazenamento | boto3, s3fs (MinIO/S3 compatível)                                 |
| Orquestração  | Apache Airflow >= 2.9                                             |
| Validação     | Pydantic >= 2.5                                                   |
| Relatórios    | fpdf2 >= 2.7, Matplotlib >= 3.8, Seaborn >= 0.13                 |
| Testes        | pytest, polars assertions                                         |
| Linting       | ruff (line-length=100, target py311)                              |

## Convenções Obrigatórias

### Estrutura de Módulos (prefixo alfabético)

```
a_configs/    → Configurações, settings, logger
b_models/     → Schemas Pydantic + Spark (StructType)
c_ingestion/  → Adaptadores de fonte de dados
d_processing/ → Lógica de cada camada (a_bronze/, b_silver/, c_gold/, d_report/)
e_validation/ → Quality checks (fail-fast assertions)
f_pipelines/  → Orquestração por camada (extract/transform/load/run)
g_storage/    → Abstrações de armazenamento
h_dags/       → DAGs do Airflow
i_notebooks/  → Notebooks de exploração
j_data/       → Dados locais por camada
k_logs/       → Logs
l_tests/      → Testes pytest
m_docs/       → Documentação
z_outputs/    → Relatórios gerados
```

### Padrões de Código

1. **Settings centralizados**: Toda config vem de `a_configs/settings.py`. NUNCA ler `os.environ` diretamente.
2. **Schemas explícitos**: Spark schemas são declarados em `b_models/schemas.py`. NUNCA usar `inferSchema=True`.
3. **Logger JSON**: Usar `from a_configs.logger import get_logger` — nunca `print()` para logs.
4. **Imports `from __future__ import annotations`**: Em TODOS os módulos.
5. **Polars como default**: Preferir Polars sobre Pandas para transformações.
6. **Pydantic v2**: Modelos com `field_validator` e `@classmethod`.
7. **Type hints**: Todos os parâmetros e retornos tipados (Python 3.11+ syntax).
8. **Docstrings**: Uma linha para funções simples, multiline para classes/complexas.

### Padrão de Pipeline (Contrato)

Toda pipeline segue o contrato:
```python
class XxxPipeline:
    def __init__(self, config: XxxPipelineConfig | None = None): ...
    def extract(self) -> pl.DataFrame: ...
    def transform(self, df: pl.DataFrame) -> pl.DataFrame: ...  # se aplicável
    def load(self, df: pl.DataFrame) -> None: ...
    def run(self) -> pl.DataFrame: ...
```

### Padrão de DAG Airflow

```python
DEFAULT_ARGS = {
    "owner": "data-engineering",
    "depends_on_past": False,
    "retries": N,
    "retry_delay": timedelta(minutes=M),
    "email_on_failure": False,
    "email_on_retry": False,
}
# Import de pipeline dentro da callable (evitar parse-time import)
# Schedule: "0 22 * * 1-5" (após fechamento B3)
# Tags: ["b3", "<layer>", "<action>"]
```

### Padrão de Teste

- Fixtures em `l_tests/conftest.py`
- DataFrames Polars como fixtures (`raw_df`, `silver_df`, etc.)
- Paths temporários com `tmp_path`
- Assertions com `assert` nativo + mensagens descritivas

### Camada Bronze

- Dados brutos IMUTÁVEIS da fonte
- Metadados adicionados: `source`, `ingested_at` (UTC)
- Particionado por `trade_date` (formato `YYMMDD_HHMM`)
- Path: `j_data/a_bronze/<source>/trade_date_YYMMDD_HHMM/data.parquet`
- Compressão: snappy

### Camada Silver

Pipeline de 6 etapas:
1. `cast_types` → tipos numéricos estritos
2. `remove_nulls` → drop nulls em campos-chave
3. `remove_invalid_prices` → preços > 0
4. `deduplicate` → 1 registro por (ticker, trade_date)
5. `calculate_daily_return` → retorno diário percentual
6. `add_metadata` → `processed_at` UTC

### Camada Gold

3 tabelas analíticas:
- `daily_metrics` (grão: ticker-data)
- `portfolio_summary` (grão: ticker)
- `monthly_returns` (grão: ticker-ano-mês)

### Quality Checks

- Fail-fast com `AssertionError`
- Rodam ANTES do write na Silver
- Checks: no_nulls, positive_prices, no_duplicates, date_range

## Tickers Padrão (Yahoo Finance format)

PETR4.SA, VALE3.SA, ITUB4.SA, BBDC4.SA, ABEV3.SA, WEGE3.SA,
RENT3.SA, MGLU3.SA, BPAC11.SA, LREN3.SA, BBAS3.SA, RADL3.SA

---
> Source: [EzequielFerreiraC/b3-data-plataform](https://github.com/EzequielFerreiraC/b3-data-plataform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-29 -->
