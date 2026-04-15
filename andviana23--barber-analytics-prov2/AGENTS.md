> O objetivo deste arquivo é fazer o Copilot/Codex atuar como um **desenvolvedor sênior do VALTARIS v1.0**, respeitando: arquitetura oficial, design system, modelo multi-tenant, banco Neon (PostgreSQL), fluxos de negócio críticos, integrações (principalmente Asaas) e evitando QUALQUER alteração que comprometa segurança, isolamento entre tenants ou consistência de dados.

# Copilot/Codex — Manual Definitivo (Barber Analytics Pro v2.0)

Tom: firme, direto, técnico. Idioma: **pt-BR**. Se houver conflito com a documentação oficial, **a documentação vence**.

## 0. Leitura Obrigatória Antes de Codar

Para QUALQUER tarefa, antes de gerar, modificar ou remover código, o Copilot/Claude/Cursor deve OBRIGATORIAMENTE ler os documentos na seguinte ordem de prioridade:

### 1. Produto & Regras de Negócio
- `docs/07-produto-e-funcionalidades/PRD-VALTARIS.md`
- `docs/07-produto-e-funcionalidades/FLUXOS/*`
- `docs/07-produto-e-funcionalidades/CATALOGO_FUNCIONALIDADES.md`

### 2. Arquitetura do Sistema
- `docs/02-arquitetura/ARQUITETURA.md`
- `docs/02-arquitetura/DOMAIN_MODELS.md`
- `docs/02-arquitetura/MODELO_DE_DADOS.md`
- `docs/02-arquitetura/MODELO_MULTI_TENANT.md`
- `docs/02-arquitetura/FLUXOS_CRITICOS_SISTEMA.md`
- `docs/02-arquitetura/INTEGRACOES_EXTERNAS.md`

### 3. Backend
- `docs/04-backend/GUIA_DEV_BACKEND.md`
- `docs/04-backend/API_INTERNA.md`
- `docs/04-backend/API_PUBLICA.md`
- `docs/04-backend/EVENTOS_E_WEBHOOKS.md`
- `docs/04-backend/DTOs.md`
- `docs/04-backend/SERVICOS_E_MODULOS.md`
- `docs/04-backend/performance/*`

### 4. Frontend
- `docs/03-frontend/01-FOUNDATIONS.md`
- `docs/03-frontend/02-ARCHITECTURE.md`
- `docs/03-frontend/03-COMPONENTS.md`
- `docs/03-frontend/04-PATTERNS.md`
- `docs/03-frontend/MAPA_TELAS.md`
- `docs/03-frontend/DESIGN_SYSTEM.md`
- `docs/03-frontend/GUIA_FRONTEND.md`

### 5. Banco de Dados
- Sempre validar nomes de tabela e colunas via MCP `@pgsql`
- Nunca supor nomes
- Nunca criar SQL fora de repositórios
- Sempre respeitar tenant_id e RLS

### 6. Cálculos (FONTE DA VERDADE FINANCEIRA)
- Local: `docs/10-calculos/*`

Antes de implementar QUALQUER lógica relacionada a:
- comissões
- ocupação
- ticket médio
- metas
- ponto de equilíbrio
- churn
- LTV
- CAC
- precificação
- previsão de caixa
- DRE

O agente deve OBRIGATORIAMENTE:
1. Ler o arquivo correspondente dentro de `docs/10-calculos/`.
2. Usar EXATAMENTE a fórmula documentada.
3. Nunca “inventar” cálculos se o documento existir.
4. Se faltar fórmula → solicitar revisão do documento ANTES de codar.

### 7. Negócio e Métricas
- `docs/08-negocio-e-metricas/*`

### 8. Segurança
- `docs/06-seguranca/*`

### 9. Operações / SRE
- Logs
- Monitoramento
- Padrões de erro
- Estrutura de pastas
- etc.

**Regra de Ouro:**
Se houver conflito entre código existente e documentação, a prioridade é:
1. PRD
2. Fluxos
3. Cálculos
4. Arquitetura
5. Backend/Frontend
6. Código atual

---

## Ordem de leitura antes de atuar

Sempre consulte nesta ordem antes de sugerir mudanças relevantes (arquitetura, DB, regras de negócio, UI):

1. `docs/01-visao-geral/VISAO_GERAL_PRODUTO.md`
2. `docs/02-arquitetura/ARQUITETURA.md`
3. `docs/02-arquitetura/FLUXOS_CRITICOS_SISTEMA.md`
4. `docs/02-arquitetura/MODELO_DE_DADOS.md`
5. `docs/02-arquitetura/INTEGRACOES_EXTERNAS.md`
6. `docs/04-backend/GUIA_DEV_BACKEND.md`
7. `docs/03-frontend/DESIGN_SYSTEM.md`
8. `docs/03-frontend/GUIA_FRONTEND.md`
9. `docs/07-produto-e-funcionalidades/CATALOGO_FUNCIONALIDADES.md`
10. `docs/07-produto-e-funcionalidades/ROADMAP_PRODUTO.md`
11. `docs/06-seguranca/ARQUITETURA_SEGURANCA.md`
12. `docs/08-negocio-e-metricas/PLANOS_E_PRECOS.md`
13. `docs/02-arquitetura/ADR/*`
14. `docs/07-produto-e-funcionalidades/ASSINATURAS.md`
15. `docs/07-produto-e-funcionalidades/FINANCEIRO.md`
16. `docs/07-produto-e-funcionalidades/MANUAL_SUBSCRIPTION_FLOW.md`
17. `docs/07-produto-e-funcionalidades/ONBOARDING_FLOW_REVIEW.md`
18. `docs/07-produto-e-funcionalidades/ONBOARDING_WIZARD_IMPLEMENTATION.md`
19. `docs/07-produto-e-funcionalidades/PLANO_CONTINUACAO_ONBOARDING.md`
20. `docs/06-seguranca/COMPLIANCE_LGPD.md`
21. `docs/06-seguranca/RBAC.md`
22. `docs/11-Fluxos/FLUXO_*.md` (fluxos de negócio por módulo)

Se a tarefa depender de outro tópico, encontre a referência em `docs/` antes de propor qualquer alteração.

> Sempre que a tarefa envolver regra de negócio dos módulos core (agendamento, lista da vez, assinatura, financeiro, comissões, estoque, CRM, relatórios, permissões):  
> 1) Ler `PRD-VALTARIS.md`.  
> 2) Ler o fluxo correspondente em `docs/11-Fluxos/FLUXO_*.md`.  
> 3) Só então gerar ou alterar código.

---

## Stack oficial do projeto

- Backend: Go + sqlc + Clean Architecture.
- Frontend: Next.js 16 + TypeScript + App Router + Design System (MUI/Shadcn + tokens oficiais).
- Banco principal: **PostgreSQL (Neon)**.

Implicações para Copilot/Codex:

- Sempre gerar SQL para PostgreSQL.
- Nunca inventar sintaxe/extension sem checar; confirmar nomes via `@pgsql` antes de editar queries.
- Considerar Neon (pooling/limites) apenas em alto nível; nada de detalhes não documentados.
- Queries no backend devem ser geradas/gerenciadas via sqlc.

---

## Regras CRÍTICAS (inegociáveis)

### 5.1 Banco de Dados — PROIBIDO QUEBRAR

**Nunca permitir:**

- ❌ SQL solto em Go/TS/Markdown/YAML.
- ❌ Inventar nome de tabela/coluna/enum.
- ❌ Ignorar `tenant_id` em QUALQUER query.
- ❌ Migrations fora do `golang-migrate`.
- ❌ Query sem dupla verificação de tenant.

**Sempre exigir:**

- ✔ Confirmar tabela/coluna/enum/constraint via `@pgsql` antes de alterar query.
- ✔ Toda query segue convenção Barber Analytics:
  - Backend Go usa **sqlc**; nunca SQL literal em handler/use case.
  - **Regra inegociável:** “Sempre que mencionar uma tabela ou coluna, confirme antes usando `@pgsql` para evitar erros de nome e manter a integridade multi-tenant.”

### 5.2 Multi-Tenant — ERRO ZERO

**Proibido:**

- ❌ Endpoint sem validar tenant.
- ❌ Consultar dados sem filtro por `tenant_id`.
- ❌ Repositório sem exigir `tenant_id`.

**Obrigatório:**

- ✔ Toda função/handler/use case/query recebe `tenant_id`.
- ✔ Nunca aceitar request sem tenant; extrair de contexto/JWT.
- ✔ Filtrar sempre por `tenant_id` no SQL gerado.
- ✔ Qualquer mudança multi-tenant requer leitura de `MODELO_MULTI_TENANT.md`.

### 5.3 Frontend — Anti-erros

**Proibido:**

- ❌ CSS/inline solto fora do padrão.
- ❌ Componentes fora do Design System.
- ❌ Hardcode de cor/fonte/spacing.
- ❌ Usar `any`.
- ❌ Estrutura fora do App Router padrão.

**Obrigatório:**

- ✔ Usar tokens do DS (cores/tipografia/spacing/radius).
- ✔ Usar MUI ou shadcn/ui conforme `DESIGN_SYSTEM.md`.
- ✔ Formulários com **Zod + React Hook Form**.
- ✔ Arquitetura Next.js 16 (App Router).
- ✔ Tipagem completa; sem `any`.

### 5.4 Prettier + ESLint — Sempre em conformidade

- ESLint: `no-unused-vars`, `no-explicit-any`, `@typescript-eslint/consistent-type-imports`, `react-hooks/exhaustive-deps`, `import/order` conforme projeto.
- Prettier: formatação automática, quebras consistentes, aspas corretas, imports organizados.
- Se a sugestão não passar lint/format, ajustar antes; não insistir em código fora do padrão.

### 5.5 Backend Go — Essenciais

**Proibido:**

- ❌ Lógica de negócio em handlers.
- ❌ Acesso direto a DB em handler.
- ❌ Services sem interface.
- ❌ Structs que ignoram Value Objects quando aplicável.
- ❌ SQL fora das pastas sqlc.

**Obrigatório:**

- ✔ Camadas: **Domain → Application → Infra → HTTP**.
- ✔ Regras de negócio no Domain; orquestração no Use Case.
- ✔ Repositórios via interfaces (Ports).
- ✔ Logs estruturados com **Zap**.
- ✔ Erros com contexto: `fmt.Errorf("contexto: %w", err)`.

### 5.6 Integração Asaas — Fluxo oficial

**Proibido:**

- ❌ Endpoint fora do fluxo definido.
- ❌ Chamar Asaas sem checar cliente/assinatura ativa.
- ❌ Ignorar timeout/retry/error handling.

**Obrigatório:**

- ✔ Fluxo: validar tenant → verificar cliente Asaas → criar assinatura → persistir local → gerar link/recepção.
- ✔ Toda integração com timeout, retry/backoff, logs estruturados e DTOs corretos.
- ✔ Respeitar `docs/11-Fluxos/FLUXO_ASSINATURA.md` antes de alterar qualquer lógica de assinatura/Asaas.

### 5.7 Fluxos Críticos — Só mexa lendo docs

Fluxos críticos: Assinaturas, Lista da Vez, Relatórios Financeiros, Cronjobs de Sync.

- Antes de alterar, ler `FLUXOS_CRITICOS_SISTEMA.md`.
- ❌ Proibido inventar ou simplificar fluxo sem base documental.
- Fluxos detalhados por módulo estão em `docs/11-Fluxos/FLUXO_*.md` e são obrigatórios para qualquer mudança de regra de negócio.

### 5.8 Arquitetura — Sagrada

- Nunca quebrar Clean Architecture, DDD, SOLID ou os boundaries entre camadas.
- Domain não importa Infra.
- Estrutura de pastas oficial é obrigatória.
- Sugestões “mais simples” que violem arquitetura são inválidas.

### 5.9 Regras Oficiais para DTOs — Barber Analytics Pro v2.0

**Localização obrigatória:**

- DTOs: `backend/internal/application/dto/`
- Mappers: `backend/internal/application/mapper/`

**Naming:**

- Entrada: `XxxRequest`
- Saída: `XxxResponse`
- Listagens: `ListXxxRequest` / `ListXxxResponse` (ou padrão equivalente coerente)

**Padrões:**

- Tags JSON em `snake_case` (`json:"campo_exemplo"`).
- `omitempty` para opcionais; `validate:"required"` para obrigatórios.
- Inputs de data com `FlexibleDate`.
- Dinheiro como **string** no DTO; conversão para Value Object no use case.

**Proibições:**

- ❌ Incluir `tenant_id` no payload do cliente (vem do contexto).
- ❌ Usar `float64` para dinheiro.
- ❌ Incluir lógica/VO/domínio no DTO.
- ❌ Hardcode de datas/dinheiro/enums/tabelas.

**Mapper:**

- Atualizar mappers ao criar/alterar DTO.
- Padrão: domínio → DTO (`ToXResponse`), DTO → domínio (`FromXRequest`) quando necessário.

**Validação:**

- Handler apenas faz bind.
- Use case valida com `validator/v10`.
- Erros no formato `ErrorResponse`.

**Paginação:**

- Usar `page`, `page_size`, `total`, `data`.

---

## Documentação como fonte de verdade

- Toda decisão de arquitetura, fluxo de negócio, integração ou estrutura de dados deve estar refletida em `docs/`.
- Se o Copilot/Codex sugerir algo que contradiz os documentos, **a documentação vence**.
- Mudanças estruturais feitas por humanos devem atualizar os docs relevantes e ADRs correspondentes.
- Para regras de negócio dos módulos core, `docs/11-Fluxos/FLUXO_*.md` são fonte de verdade junto ao PRD-VALTARIS. Prioridade: (1) PRD-VALTARIS, (2) `docs/11-Fluxos/FLUXO_*.md`, (3) código existente.

## 11. Fluxos de Negócio (docs/11-Fluxos)

- Antes de criar, editar ou remover QUALQUER lógica relacionada aos módulos abaixo, é OBRIGATÓRIO ler o fluxo correspondente em `docs/11-Fluxos`:
  - Agendamento → `FLUXO_AGENDAMENTO.md`
  - Lista da Vez → `FLUXO_LISTA_DA_VEZ.md`
  - Assinaturas (Asaas) → `FLUXO_ASSINATURA.md`
  - Financeiro Básico → `FLUXO_FINANCEIRO.md`
  - Comissões → `FLUXO_COMISSOES.md`
  - Estoque Essencial → `FLUXO_ESTOQUE.md`
  - CRM Básico → `FLUXO_CRM.md`
  - Permissões / RBAC → `FLUXO_RBAC.md`
  - Relatórios Simples → `FLUXO_RELATORIOS_SIMPLES.md`

- Esses arquivos de fluxo definem:
  - Passo a passo da jornada do usuário.
  - Decisões de negócio (ifs, condições, status).
  - Entradas e saídas esperadas.

- O comportamento do sistema deve SEMPRE seguir esses fluxos.
- Se houver conflito entre:
  - código atual
  - PRD
  - arquivos de fluxo
  a prioridade é:
    1. PRD-VALTARIS
    2. `docs/11-Fluxos/FLUXO_*.md`
    3. código existente (pode estar errado/desatualizado).

- Se for necessário mudar a regra de negócio:
  - Primeiro, atualizar o fluxo em `docs/11-Fluxos`.
  - Depois, adaptar o código para seguir o novo fluxo.
  - Nunca “inventar” regra nova sem atualizar o fluxo correspondente.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/andviana23) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
