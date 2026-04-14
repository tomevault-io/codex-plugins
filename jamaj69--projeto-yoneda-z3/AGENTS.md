# Projeto Yoneda-Z3: Instruções de Contexto

## 🎯 Visão Geral do Projeto

Sistema híbrido de otimização para **Job Shop Scheduling Problem (JSSP)** que combina:
- **Haskell (servidor REST)**: Pipeline de 3 fases — MWR+SPT → SBP → Yoneda-fused N2/N5/N7
- **Python + Z3/OR-Tools**: Solvers exatos para soluções ótimas globais

### Arquitetura

```
Python → POST /validate → Haskell (Phase 1a: MWR → 1b: SBP → 2: N2/N5/N7 → 3: analysis)
```

## 📊 Estado Atual (v0.4.0)

### Funcionalidades Implementadas

1. **Servidor Haskell** (`app-haskell/src/Main.hs`)
   - Porta 3000, endpoint `/validate`
   - **Phase 1a**: MWR+SPT list scheduling
   - **Phase 1b**: Shifting Bottleneck (Schrage + Carlier B&B)
   - **Phase 2**: Yoneda-fused neighborhoods N2/N5/N7 with `greedySweep` and `refinementPipeline`
   - **Phase 3**: Bottleneck analysis (slack, critical path, utilization)
   - Retorna: `hints`, `makespan_heuristic`, `makespan_sbp`, `makespan_refined`, `refined_starts`, `slacks`, `critical_path`, `critical_machines`, `machine_utilization`

2. **Cliente Python** (`script-python/`)
   - `main.py`: Exemplo básico 4×3
   - `example_usage.py`: Integração Haskell+Z3 com benchmarks
   - `solve_ortools.py`: OR-Tools CP-SAT solver
   - `instance_loader.py`: Carrega 242 instâncias clássicas
   - `solve_with_bottlenecks.py`: Análise de gargalos
   - `validate_solution.py`, `verify_ortools.py`: Validação de soluções
   - `learn_from_z3.py`: Análise comparativa heurística vs Z3
   - `debug_z3.py`: Debug de comportamento Z3

3. **Benchmarks** (`instances/`)
   - 242 instâncias de 8 datasets clássicos
   - FisherThompson1963, Lawrence1984, Taillard1993, AdamsBalasZawack1988, etc.

### Resultados de Performance (Haskell heurística pura)

| Instância | Dim | MWR | SBP | Refined | BKS | Gap | Time |
|-----------|-----|-----|-----|---------|-----|-----|------|
| ft06 | 6×6 | 69 | 60 | 60 | 55 | +9.1% | <0.1s |
| la01 | 10×5 | 880 | 666 | **666** | 666 | **OPT** | <0.1s |
| abz5 | 10×10 | 1451 | 1334 | 1312 | 1234 | +6.3% | 0.2s |
| abz9 | 10×10 | 1132 | 849 | 801 | 661 | +21.2% | 2.5s |
| dmu10 | 20×20 | 4765 | 4143 | 3621 | n/a | — | 4.3s |
| ta71 | 100×20 | 8010 | 5930 | 5886 | 5464 | +7.7% | 24s |

**Evolução histórica:**
- v0.1.0 (toposort): 6446 em abz5
- v0.2.0 (MWR+SPT): 1451 em abz5 (77% melhor)
- v0.3.0 (+ bottlenecks): Mesma qualidade, com análise detalhada
- v0.4.0 (SBP + N2/N5/N7): 1312 em abz5, la01 OPT, ta71 em 24s

## 🔧 Como Desenvolver

### Iniciar Servidor Haskell

```bash
stack build
stack run  # Porta 3000
```

### Executar Scripts Python

```bash
# Exemplo básico
python script-python/main.py

# Com benchmark
python script-python/example_usage.py instances/FisherThompson1963/ft06.txt

# Com análise de gargalos
python script-python/solve_with_bottlenecks.py instances/AdamsBalasZawack1988/abz5.txt
```

### Rodar Testes

```bash
stack test
```

## 🧠 Conceitos-Chave do Domínio

### Job Shop Scheduling Problem (JSSP)

- **Jobs**: Conjuntos de tarefas com precedência (j1, j2, ..., jn)
- **Máquinas**: Recursos compartilhados (m1, m2, ..., mk)
- **Constraints**:
  - Cada tarefa em UMA máquina específica
  - Ordem de execução dentro do job (precedência)
  - Máquina processa UMA tarefa por vez
- **Objetivo**: Minimizar makespan (tempo total)

### Heurística MWR+SPT (Phase 1a)

**Most Work Remaining (MWR)** + **Shortest Processing Time (SPT)** list scheduling:
```haskell
priority task = (negate remaining_work, duration task, task_id)
sortBy (comparing priority) tasks
```

### Shifting Bottleneck Procedure (Phase 1b)

- **Schrage heuristic** para subproblemas `1|r_j|max(C_j+q_j)`
- **Carlier B&B** com node budget (budget=0 = pure Schrage, empiricamente suficiente)
- Decomposição iterativa: bottleneck machine → fix ordering → re-optimize

### Yoneda-Fused Neighborhoods (Phase 2)

```haskell
newtype Yoneda f a = Yoneda { runYoneda :: forall b. (a -> b) -> f b }
-- O(1) fmap, composição na continuação, lowerYoneda aplica tudo em 1 passagem
```

- **N2**: Adjacent swap on critical path (Nowicki & Smutnicki 1996)
- **N5**: Block rotation (van Laarhoven et al. 1992)
- **N7**: Task reinsertion at all positions (Dell'Amico & Trubian 1993)
- **greedySweep**: carry-forward `foldl'` que permite cadeias de swaps dependentes
- **refinementPipeline**: left-fold `[(budget, Neighborhood)]`

### Análise de Gargalos (Phase 3)

- Forward/backward pass no grafo disjuntivo para EST/LST
- `Slack = LST - EST` (zero = caminho crítico)
- `Utilização = Tempo_Trabalhado / Makespan`

### Solvers Exatos (Python)

- **Z3**: Hints usados apenas como REFERÊNCIA (não constraints!)
- **OR-Tools CP-SAT**: `NewIntervalVar` + `AddNoOverlap` (cold-start, sem warm-start)

## 📁 Estrutura de Arquivos

```
.
├── app-haskell/
│   └── src/Main.hs              # Servidor + pipeline completo
├── src/
│   ├── Types.hs                 # TaskReq, TaskRes, ValidationResponse
│   ├── Run.hs                   # Lógica de execução
│   └── Util.hs                  # Utilitários
├── script-python/
│   ├── main.py                  # Exemplo básico
│   ├── example_usage.py         # Integração Haskell+Z3
│   ├── solve_ortools.py         # OR-Tools CP-SAT solver
│   ├── instance_loader.py       # Parser de benchmarks
│   ├── solve_with_bottlenecks.py # Análise de gargalos
│   ├── validate_solution.py     # Validação de soluções
│   ├── verify_ortools.py        # Verificação OR-Tools
│   ├── learn_from_z3.py         # Análise comparativa
│   └── debug_z3.py              # Debug Z3
├── instances/                   # 242 benchmarks
├── docs/
│   ├── BOTTLENECK_ANALYSIS.md   # Análise de gargalos
│   ├── HEURISTIC_IMPROVEMENTS.md # Evolução da heurística
│   ├── FEEDBACK_LEARNING.md     # Sistema de aprendizado
│   ├── INSTANCE_LOADER.md       # Guia de benchmarks
│   └── WHY_NOT_OPTIMAL.md       # Z3 non-determinism
├── ROADMAP.md                   # Planejamento v0.5+
├── CHANGELOG.md                 # Histórico de versões
└── README.md                    # Documentação principal
```

## 🎯 Próximos Passos (v0.5.0)

### 1. IntMap Adjacency (ALTA PRIORIDADE)
Replace `algebraic-graphs` with `IntMap IntSet` for incremental graph updates in `evalCandidate`.
Expected: ta71 from 24s → ~5s.

### 2. Tabu Search over N2/N5 (ALTA PRIORIDADE)
Short-term memory to escape local minima. Expected: abz5 1312→~1260.

### 3. SBP Graph Caching (MÉDIA PRIORIDADE)
Thread graph through SBP iterations to avoid redundant rebuilds.

### 4. Better Schrage Tie-Breaking (MÉDIA PRIORIDADE)
Composite tie-breaker to prevent SBP convergence oscillation.

### Empirical Findings (tested and rejected)
- ❌ Carlier B&B budget > 0: Schrage already optimal per subproblem
- ❌ SBP convergence loop: oscillation due to Schrage tie-breaking
- ❌ OR-Tools warm-start: CP-SAT cold-start is empirically better

## ⚙️ Configurações Importantes

### Haskell

**Stack** (stack.yaml):
```yaml
resolver: lts-22.28
packages:
- .
```

**Dependências** (package.yaml):
```yaml
dependencies:
- scotty              # Web framework
- algebraic-graphs    # Validação de ciclos
- aeson               # JSON
- containers          # Data structures
```

### Python

**Versão**: 3.11+

**Dependências**:
```bash
pip install z3-solver requests matplotlib
```

**Setup time** (em example_usage.py):
```python
SETUP_TIME = 0  # Comparação com literatura
```

## 📚 Referências Técnicas

### Papers e Livros

- **JSSP**: Garey & Johnson "Computers and Intractability" (1979)
- **SBP**: Adams, Balas & Zawack "The Shifting Bottleneck Procedure" (1988)
- **Schrage**: Schrage "Greedy heuristic for single-machine scheduling" (1984)
- **Carlier**: Carlier "Branch-and-bound for 1|r_j|Lmax" (1982)
- **N2**: Nowicki & Smutnicki "i-TSAB" (1996)
- **N5**: van Laarhoven, Aarts & Lenstra (1992)
- **N7**: Dell'Amico & Trubian (1993)
- **Local Search**: Aarts & Lenstra "Local Search in Combinatorial Optimization" (1997)
- **Z3**: Bjørner & Phan "νZ - Maximal Satisfaction with Z3" (2014)

### Benchmarks

- **OR-Library**: http://people.brunel.ac.uk/~mastjjb/jeb/orlib/jobshopinfo.html
- **Best Known Solutions**: http://jobshop.jjvh.nl/

## 🔍 Comandos Úteis para Debug

### Ver estado do servidor

```bash
curl -X POST http://localhost:3000/validate \
  -H "Content-Type: application/json" \
  -d '[{"id_t":1,"job_id":1,"machine_id":1,"duration":3,"next_t":2,"prev_t":null}]'
```

### Comparar heurística vs Z3

```bash
python script-python/solve_with_bottlenecks.py instances/AdamsBalasZawack1988/abz5.txt
```

### Testar Z3 isolado (sem hints)

```bash
python script-python/debug_z3.py
```

### Profile Haskell

```bash
stack build --profile
stack run --profile -- +RTS -p
cat haskell-engine-exe.prof
```

## 💡 Dicas de Desenvolvimento

### Ao modificar heurística:

1. Testar em ft06 primeiro (6×6, rápido)
2. Depois em abz5 (10×10, benchmark principal)
3. Comparar com best known: ft06=55h, abz5=1234h
4. Atualizar CHANGELOG.md

### Ao adicionar análise:

1. Adicionar campo em `ValidationResponse` (Types.hs)
2. Implementar função de análise (Main.hs)
3. Atualizar endpoint `/validate`
4. Criar script Python correspondente
5. Documentar em docs/

### Ao testar:

1. Sempre rodar `stack test` antes de commit
2. Testar em pelo menos 3 instâncias (ft06, la01, abz5)
3. Verificar se gap vs optimal não piorou
4. Atualizar tabela de resultados no README.md

## 🚀 Workflow de Commit

```bash
# 1. Build e test
stack build && stack test

# 2. Verificar mudanças
git status
git diff

# 3. Commit semântico
git add -A
git commit -m "tipo: descrição

- Detalhe 1
- Detalhe 2"

# Tipos: feat, fix, docs, refactor, perf, test

# 4. Push
git push origin main
```

## 📞 Contato

**Autor**: Jose Augusto M de Andrade Jr  
**Email**: jamaj@jamaj.com.br  
**Repositório**: https://github.com/jamaj69/projeto-yoneda-z3

---

## 🎬 Checklist Rápido ao Retomar

Ao voltar ao projeto após pausa:

- [ ] Ler CHANGELOG.md para ver última versão
- [ ] Verificar ROADMAP.md para próximos passos
- [ ] Rodar `stack build && stack run` (servidor na porta 3000)
- [ ] Testar com `python script-python/solve_with_bottlenecks.py instances/AdamsBalasZawack1988/abz5.txt`
- [ ] Verificar se resultado ainda é ~1312 (refined) e ~1334 (SBP)
- [ ] Escolher feature do ROADMAP.md para implementar

**Status de testes esperado**:
- ft06: refined ~60, BKS=55
- la01: refined ~666 OPT, BKS=666
- abz5: refined ~1312, BKS=1234
- abz9: refined ~801, BKS=661
- ta71: refined ~5886, BKS=5464

Se os resultados mudarem significativamente, algo regrediu! 🚨

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jamaj69) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
