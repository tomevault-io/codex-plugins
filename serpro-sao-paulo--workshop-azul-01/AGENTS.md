# Instruções do GitHub Copilot — Workshop de Modernização de Legado

> Contexto, stack, convenções e regras que se aplicam a todo o repositório. Confie nestas instruções; pesquise no código só quando estiverem incompletas ou incorretas.

## Contexto do Projeto

Modernização do legado **SIFAP** (Sistema de Fiscalização e Administração de Pagamentos) — Natural/Adabas, 29 anos — para Java 21 + Next.js 15. Código legado em [`01-arqueologia/legado-sifap/`](../01-arqueologia/legado-sifap/) (15 programas `.NSN` + 4 DDMs). Dois níveis de agentes: persona-kit por pessoa + agente de estágio por equipe ([`06-agentes-de-estagio/README.md`](../06-agentes-de-estagio/README.md)).

## Ferramentas Aprovadas — Somente Estas

Toolchain fixa; misturar ferramentas quebra a rastreabilidade spec → code → test e as demos.

- **VS Code** / VS Code Insiders — editor único (as extensões e os kits Copilot do repositório assumem isso)
- **GitHub Copilot** (Ask + Plan + Agent) — IA principal; Copilot Workspace permitido para Issue → PR
- **GitHub Copilot CLI** *(opcional)* — tarefas em terminal
- **GitHub Spec-Kit** (`Specify CLI` + `/speckit.*`) — Spec-Driven Development
- **GitHub** (Issues, PRs, Actions, Projects) — fonte da verdade
- **Docker / Docker Compose** — paridade local · **Terraform** — IaC (Azure provider)

**Não use** outros assistentes de IA (Cursor, Windsurf, Codex, Cline, Continue, Aider, Codeium, Tabnine), IDEs alternativos (IntelliJ, Eclipse, Neovim), UIs web de chat para gerar código, nem frameworks SDD alternativos (Kiro etc.).

## Stack-Alvo

- **Backend:** Java 21 + Spring Boot 3.3 + JPA/Hibernate + PostgreSQL 16
- **Frontend:** Next.js 15 (App Router) + TypeScript 5 (strict) + Tailwind CSS + shadcn/ui
- **Containers:** Docker + Docker Compose
- **IaC:** Terraform (Azure provider ~> 3.x)
- **CI/CD:** GitHub Actions
- **Testing:** JUnit 5 + Testcontainers (backend); Vitest + Testing Library (frontend)

## Regras de Geração de Código

### Java
- Use recursos do Java 21: records para DTOs, sealed interfaces para uniões discriminadas, pattern matching, virtual threads
- Use `Optional` corretamente — nunca retorne `null` de métodos públicos
- `@Transactional` somente na camada de service, nunca em repositories
- Valide entradas na camada de controller com `@Valid` + Bean Validation
- Nomes de classes em inglês; comentários em inglês
- Testes unitários são obrigatórios para lógica de negócio
- Nunca exponha dados sensíveis (CPF, valores de benefício) em logs — mascare-os

### TypeScript / Next.js
- `strict: true` em `tsconfig.json` — sem exceções
- Use server actions para mutations; nunca exponha secrets em client components
- Prefira `async/await` a cadeias `.then()`
- Somente named exports — sem default exports em arquivos de componentes

### REST APIs
- Convenção de path: `/api/v1/{resource}`
- Use verbos HTTP corretamente (`GET`, `POST`, `PUT`, `PATCH`, `DELETE`)
- Retorne status codes apropriados (`201` para criação, `204` para sem conteúdo, `409` para conflito)
- Todos os endpoints devem ter annotations OpenAPI/Swagger

### Terraform
- Todo recurso deve ter `tags` incluindo `project`, `environment`, `owner`
- Secrets somente via `azurerm_key_vault_secret` — nunca em `locals` ou `variables`
- Um módulo por área de serviço Azure (networking, compute, database, monitoring)
- `terraform fmt` e `terraform validate` devem passar antes do commit

## Regras de Segurança (OWASP Top 10)

- Valide entradas em toda fronteira do sistema
- Nunca faça hardcode de secrets, API keys ou credenciais
- Consultas SQL somente via JPA/JPQL — sem concatenação de strings
- CORS configurado explicitamente — sem wildcard `*` em produção
- Autenticação via OAuth2/JWT (Spring Security no backend)
- Todos os recursos Azure usam Managed Identity para autenticação serviço-a-serviço

## Spec-Driven Development (Spec-Kit)

- Todo requisito usa **notação EARS** (Easy Approach to Requirements Syntax)
- Todo requisito tem um **REQ-ID** único no formato `REQ-NNN`
- **Todo requisito carrega uma linha `source_legacy:`** apontando para `01-arqueologia/legado-sifap/natural-programs/*.NSN`, `01-arqueologia/legado-sifap/adabas-ddms/*.ddm` ou `[GREENFIELD] + justificativa`. O job de CI `legacy-traceability` rejeita PRs que violam isso. Consulte [`01-arqueologia/LEGACY-EXPLORATION-CHECKLIST.md`](../01-arqueologia/LEGACY-EXPLORATION-CHECKLIST.md).
- Testes rastreiam para REQ-IDs por comentários inline
- Estratégia de branch: `spec/<NNN>-<feature>` → `develop` → `main` (sem `stage`; ver [`00-GIT-WORKFLOW.md`](../00-GIT-WORKFLOW.md))
- Antes de escrever EARS no Estágio 2, o par DEVE ter lido os programas Natural atribuídos (HARD GATE — ver checklist acima)

## Regras Rígidas — Não Faça Isto

- ❌ Não gere código da nova aplicação sem antes ler o legado em `01-arqueologia/legado-sifap/` e ter um REQ-ID com `source_legacy:`
- ❌ Não adicione dependências sem justificativa em um ADR
- ❌ Não escreva testes depois do fato — escreva-os enquanto implementa
- ❌ Não exponha secrets em mensagens de commit, logs ou descrições de PR
- ❌ Não faça merge em `main` sem pelo menos uma revisão entre pares
- ❌ Não pule as conversas guiadas de passagem nas transições de estágio ([`00-TEAM-FLOW.md`](../00-TEAM-FLOW.md))

## Referências

- 3 modos do Copilot (Ask · Plan · Agent): [`09-cheat-sheets/copilot-3-modes.md`](../09-cheat-sheets/copilot-3-modes.md)
- Persona kits (já incluídos no `.github/` deste repositório): [`05-personas/`](../05-personas/) · Agentes de estágio: [`06-agentes-de-estagio/`](../06-agentes-de-estagio/)
- A nova aplicação (backend Java + frontend Next.js + infra) é criada pelo time com o Spec-Kit no Estágio 3 — não há código de aplicação pronto neste repositório
- Spec-Kit SDD: <https://github.com/github/spec-kit>

<!-- SPECKIT START -->
For additional context about technologies to be used, project structure,
shell commands, and other important information, read the current plan
<!-- SPECKIT END -->

---
> Source: [serpro-sao-paulo/workshop-azul-01](https://github.com/serpro-sao-paulo/workshop-azul-01) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-23 -->
