# Memória do Projeto — MBA TCC Análise de Sentimento

## Visão Geral

Projeto de conclusão de curso (TCC) que coleta reportagens do portal G1, identifica menções a bairros de São Paulo via NER (Named Entity Recognition) e classifica o sentimento associado a cada bairro (positivo, neutro, negativo) usando modelos transformer.

**Stack**: Python 3.11+ · UV (gerenciador de pacotes) · PostgreSQL · SQLModel · spaCy · Hugging Face Transformers · PyTorch

---

## Estrutura de Módulos

| Módulo | Responsabilidade |
|---|---|
| `common/` | Modelos ORM (SQLModel), configuração do banco e utilitários compartilhados |
| `extractor/` | Web crawler que lê o sitemap do G1, respeita robots.txt, extrai artigos e armazena no banco |
| `wrangler/` | Pré-processamento de texto (normalização, remoção de URLs, unicode, etc.) |
| `modeling/ner/` | NER rule-based com spaCy PhraseMatcher para bairros de SP (com variações e validação geográfica) |
| `modeling/models/` | Analyzers de sentimento (DistilBERT, BERTimbau), model registry e factory |
| `modeling/sentiment/` | Processamento em batch, agregação e CLI de sentimento |
| `modeling/tuning/` | Avaliação, grid search de hiperparâmetros, dataset builder e fine-tuning |
| `modeling/constants.py` | Constantes compartilhadas: padrões NER, regras neutras, palavras fortes, falsos positivos |
| `tests/` | Testes unitários e de integração (pytest) |
| `iac/` | Docker Compose + scripts SQL de inicialização do PostgreSQL |
| `scripts/` | Utilitários (backup do banco) |

---

## Como Rodar

### Pré-requisitos

```bash
# Instalar UV (se necessário)
curl -LsSf https://astral.sh/uv/install.sh | sh

# Instalar dependências
uv sync

# Subir o banco PostgreSQL
cd iac && docker-compose up -d
```

Credenciais ficam no arquivo `.env` na raiz do projeto.

### CLIs Disponíveis

#### Extractor (crawler G1)

```bash
# Extrair artigos do sitemap
uv run python -m extractor crawl

# Extrair com limite
uv run python -m extractor crawl --limit 500
```

#### Main (NER)

```bash
# Rodar NER nos artigos extraídos
uv run python main.py ner --limit 1000
```

#### Sentiment Analysis

```bash
# Analisar sentimento (todos os níveis)
uv run python -m modeling.sentiment analyze all

# Analisar por nível específico
uv run python -m modeling.sentiment analyze zone|subprefecture|neighborhood

# Com modelo alternativo (feature toggle)
uv run python -m modeling.sentiment analyze all --model-key bertimbau --fine-tuned-path models/bertimbau-sentiment/best

# Estatísticas
uv run python -m modeling.sentiment stats [--format json]

# Breakdown por zona ou subprefeitura
uv run python -m modeling.sentiment breakdown zone|subprefecture

# Testar com texto livre
uv run python -m modeling.sentiment test --text "Texto aqui"

# Listar modelos disponíveis
uv run python -m modeling.sentiment models

# Popular tabela de agregações
uv run python -m modeling.sentiment aggregate --period-type monthly
```

#### Tuning & Avaliação

```bash
# Avaliar com mock data (8 test cases)
uv run python -m modeling.tuning evaluate [-v|-vv]

# Grid search de hiperparâmetros
uv run python -m modeling.tuning tune --quick|--full

# Analisar test case específico
uv run python -m modeling.tuning analyze <nome>

# Listar test cases
uv run python -m modeling.tuning list

# Construir dataset de avaliação a partir do banco
uv run python -m modeling.tuning build-dataset --limit 100 --strategy random|balanced|recent

# Avaliar contra dataset JSON rotulado manualmente
uv run python -m modeling.tuning evaluate-json <caminho.json>

# Fine-tuning do BERTimbau
uv run python -m modeling.tuning fine-tune --dataset data/eval_dataset.json --epochs 5
```

#### Testes

```bash
# Rodar todos os testes (exceto slow)
uv run python -m pytest tests/ -v -k "not slow"

# Rodar apenas testes do modeling
uv run python -m pytest tests/modeling/ -v

# Rodar apenas testes do extractor
uv run python -m pytest tests/extractor/ -v

# Com cobertura
uv run python -m pytest tests/ --cov=modeling --cov=common
```

---

## Modelos ORM (common/models.py)

| Tabela | Descrição |
|---|---|
| `SitemapEntry` | URLs do sitemap G1 com status de extração |
| `Article` | Artigos extraídos (título, conteúdo, data, autor, URL) |
| `Neighborhood` | Bairros de SP com zona e subprefeitura |
| `EntityMention` | Menções de bairros encontradas nos artigos (NER) |
| `SentimentAnalysisResult` | Resultado de sentimento por menção (label, score, raw_scores) |
| `SentimentAggregation` | Agregações de sentimento por período e nível geográfico |

---

## Pipeline de Sentimento (Hybrid: Rules + Model)

```
Texto do artigo
  │
  ├─ STEP 1: Validação geográfica (rule-based)
  │   └─ Detecta se o bairro está no contexto de outra cidade → DISCARD
  │
  ├─ STEP 2: Validação semântica (rule-based)
  │   └─ Verifica se a menção é referência a local e não outro significado → DISCARD
  │
  ├─ STEP 3: Extrai sentenças relevantes (que mencionam o bairro)
  │
  ├─ STEP 3.5: Regras neutras (pre-model)
  │   └─ Padrões fortes/fracos de conteúdo factual → NEUTRAL
  │
  ├─ STEP 4: Inferência do modelo transformer
  │
  └─ STEP 5: Correção pós-modelo
      └─ Se modelo incerto + padrões neutros presentes → override NEUTRAL
```

---

## Model Registry (Feature Toggle)

Dois modelos registrados:

| Key | Modelo | Status |
|---|---|---|
| `distilbert` (default) | `lxyuan/distilbert-base-multilingual-cased-sentiments-student` | Pronto para uso |
| `bertimbau` | `neuralmind/bert-base-portuguese-cased` | Requer fine-tuning |

Usar via CLI: `--model-key distilbert|bertimbau`
Usar via código: `create_analyzer(model_key="bertimbau", fine_tuned_path="...")`

A lógica de `skip_processed` é **model-aware**: resultados do DistilBERT não impedem reprocessamento com BERTimbau (e vice-versa). Ambos os resultados coexistem na tabela `sentiment_analysis_results`, diferenciados pela coluna `model_name`.

Novos modelos podem ser registrados com `register_model(ModelConfig(...))`.

---

## Thresholds de Confiança

| Constante | Valor Default | Descrição |
|---|---|---|
| `CONFIDENCE_THRESHOLD_HIGH` | 0.70 | Acima = alta confiança |
| `CONFIDENCE_THRESHOLD_LOW` | 0.45 | Abaixo = baixa confiança |
| `SCORE_DIFFERENCE_THRESHOLD` | 0.15 | Diferença mínima entre top-2 scores |

Estes são configuráveis por instância do `SentimentAnalyzer` e pelo grid search do tuner.

---

## Convenções de Código

- **Gerenciamento de pacotes**: UV (`uv sync`, `uv run`)
- **Type hints**: obrigatório em todo código Python
- **Strings**: f-strings sempre
- **ORM**: SQLModel (nunca regras no banco)
- **HTTP**: requests
- **Scraping**: BeautifulSoup + lxml
- **Idempotência**: funções devem retornar a mesma saída para a mesma entrada
- **Testes**: pytest, testar todo componente novo ou modificado
- **Credenciais**: `.env` na raiz (nunca hardcoded)
- **Dados**: extrair a partir de 2019-01-01

---

## Banco de Dados

- PostgreSQL via Docker Compose (`iac/docker-compose.yml`)
- Scripts de init em `iac/postgres/init/`
- Nenhuma regra de negócio no banco — toda lógica em Python
- Connection string no `.env`

---

## Arquivos Importantes

| Arquivo | Propósito |
|---|---|
| `AGENTS.md` | Contexto completo do projeto, premissas, constantes (bairros), requisitos detalhados |
| `.github/copilot-instructions.md` | Este arquivo — memória operacional para agentes |
| `pyproject.toml` | Dependências e configs do projeto |
| `docs/MODELING_TECHNIQUES.md` | Documentação das técnicas de NER e sentimento |
| `docs/TUNING_RESULTS.md` | Resultados de tuning de hiperparâmetros |
| `tests/mock/` | Arquivos de texto para test cases de sentimento (positive-1.txt, negative-1.txt, etc.) |

---
> Source: [ynfialho/mba-tcc-sentiment-analysis](https://github.com/ynfialho/mba-tcc-sentiment-analysis) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-13 -->
