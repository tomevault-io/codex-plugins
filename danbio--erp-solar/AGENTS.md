# Instruções para agentes de IA

## Visão geral do projeto

ERP para empresa de energia solar, cobrindo operação ponta a ponta.

Fluxo oficial do negócio:

`Lead → Venda → Engenharia → Faturamento → Instalação → Pós-venda/O&M`

> Importante: **instalação só começa após confirmação da entrada paga**.

Plano completo: [project-plan.md](project-plan.md)

---

## Regras não negociáveis

1. Arquitetura do MVP é **templates-first** com Django + HTMX.
2. Não avançar para `ReadyToInstall` sem `payment_entry_status = confirmed`.
3. Não sugerir SPA/DRF/Celery como obrigatórios sem ADR aprovado.
4. Toda mudança de status crítica deve ser auditável.

---

## Arquitetura escolhida

- Backend: Django 6 (templates, forms nativos, auth por sessão).
- Interatividade: HTMX 2.x para ações parciais.
- Estilo: Tailwind CSS 4 via CLI standalone (sem Node/npm).
- Banco: PostgreSQL.
- Cache/filas leves: Redis + Django Tasks.
- Pagamentos: gateway com suporte a webhook e conciliação de entrada.

Stack essencial (MVP): Django, gunicorn, psycopg, decouple, whitenoise, redis, mercadopago, Pillow, tailwindcss (CLI standalone).

---

## Domínio e nomenclatura obrigatória

Use nomes consistentes em código e documentação:

- `lead`, `opportunity`, `proposal`, `contract`
- `site_survey`, `engineering_project`, `homologation`
- `invoice`, `payment_entry_status`
- `work_order`, `ready_to_install`, `commissioning`
- `ticket`, `maintenance_plan`, `generation_reading`

Estados recomendados:

- `Lead`, `Qualified`, `ProposalSent`, `Won`
- `Engineering`, `Invoiced`, `ReadyToInstall`, `Installed`, `OMActive`

---

## Fluxo de desenvolvimento

### Prioridade de implementação

1. `core` + `users`
2. `crm` + `sales`
3. `engineering`
4. `billing` (**gate da entrada paga**)
5. `installation`
6. `support_om`

### Diretrizes técnicas

- Preferir views baseadas em classe onde fizer sentido.
- Preferir Django Forms para validação server-side.
- Usar HTMX para atualizar tabelas/cards/status sem SPA.
- Evitar JavaScript custom fora do necessário para UX.

---

## Tailwind CSS 4 — Convenções obrigatórias

### Setup

- Binário standalone: `tailwindcss.exe` (Windows) na raiz ou em `bin/`.
- Sem Node.js, sem npm, sem `tailwind.config.js`.
- Compilação: `./tailwindcss -i static/css/input.css -o static/css/output.css --watch --minify`.
- Nunca editar `output.css` manualmente — é gerado automaticamente.

### Arquivo `input.css` (CSS-first config — single source of truth)

A paleta de cores do projeto está definida **exclusivamente** em `static/css/input.css` no bloco `@theme`.

> **Nunca hardcode valores hex nos templates ou documentação.**
> Sempre consulte `static/css/input.css` para os tokens atuais.

Tokens disponíveis (nomes das classes Tailwind):

- **Primárias:** `solar-300` (claro), `solar-500` (principal), `solar-600` (hover), `solar-700` (pressed)
- **Secundárias:** `ocean-500`, `ocean-600`, `ocean-700`
- **Neutras:** `slate-title` (títulos), `slate-text` (corpo), `slate-border` (bordas), `slate-muted`, `slate-light`, `slate-bg`
- **Semânticas:** `success`, `danger`, `danger-dark`, `info`, `info-dark`, `warning`, `warning-light`

### Regras de uso em templates

- Usar utilities nativas do Tailwind (`bg-solar-500`, `text-slate-title`, etc.).
- Criar componentes reutilizáveis com `@utility` apenas se repetir 3+ vezes.
- Dark mode com `@variant dark { ... }` quando necessário.
- Responsividade com breakpoints padrão (`sm:`, `md:`, `lg:`, `xl:`).
- Nunca misturar CSS custom inline com Tailwind sem justificativa.

### `@source` — Detecção de classes

Tailwind 4 varre automaticamente arquivos listados em `@source`.
Para Django, apontar para a pasta de templates:

```css
@source "../../templates"; /* templates globais */
@source "../../apps/**/templates"; /* templates por app */
```

---

## Ícones Intelbras

SVGs da marca em `static/fonts/icons/` (91 ícones, variantes filled e `_linha` outline).

Renderizar **sempre** via template tag — nunca `<img>`, inline SVG manual ou emoji:

```html
{% load icons %} {% icon "editar" %} {% icon "confirmacao" css_class="h-6 w-6
text-success" %} {% icon "excluir_linha" css_class="h-4 w-4 text-danger" %} {%
icon "pesquisar" css_class="h-5 w-5" aria_label="Pesquisar" %}
```

O `fill="currentColor"` é injetado automaticamente — colorir com `text-*` do Tailwind.
Catálogo completo: `static/fonts/icons/` ou `django-templates.instructions.md`.

---

## HTMX 2.x — Convenções obrigatórias

### Setup

- HTMX servido como static file local: `static/js/htmx.min.js` (14kb).
- Incluir no `base.html` com `{% static 'js/htmx.min.js' %}`.
- Não usar CDN em produção.

### Padrões de uso com Django

- `hx-get` / `hx-post` para ações parciais sem reload de página.
- `hx-target` para definir onde o fragmento HTML será inserido.
- `hx-swap="innerHTML"` (padrão) ou `outerHTML` para substituição completa.
- `hx-trigger` para eventos customizados (ex: `change`, `submit`, `revealed`).
- `hx-confirm` para ações destrutivas (ex: excluir, cancelar contrato).
- `hx-boost="true"` em `<body>` ou `<main>` para navegação AJAX transparente.

### CSRF com HTMX

Incluir no `base.html`:

```html
<body hx-headers='{"X-CSRFToken": "{{ csrf_token }}"}'></body>
```

### Partials (fragmentos)

- Views HTMX retornam fragmentos HTML (sem `base.html`).
- Organizar em `templates/<app>/partials/` (ex: `lead_row.html`, `status_badge.html`).
- Usar `request.headers.get('HX-Request')` para detectar chamada HTMX na view.

### Anti-padrões

- ❌ Não usar HTMX para orquestrar lógica de negócio complexa no frontend.
- ❌ Não duplicar validação — manter server-side com Django Forms.
- ❌ Não usar `hx-vals` para dados sensíveis (tokens, IDs críticos).

---

## Segurança e LGPD

- Minimização de dados pessoais e retenção definida.
- Registro de base legal e finalidade para dados sensíveis.
- Sessões seguras, CSRF habilitado e HTTPS obrigatório em produção.
- Trilhas de auditoria para alteração de status de negócio.

---

## MCPs (Model Context Protocol) — Ferramentas externas

### Context7 — Documentação live

Busca documentação atualizada de bibliotecas em tempo real. Configurado em `.vscode/mcp.json`.

**Quando usar**: quando houver dúvida sobre API, sintaxe ou comportamento.  
**Quando NÃO usar**: operações triviais onde o agente já tem confiança.

**Workflow**:

1. Resolver ID da lib: `resolve-library-id` com nome (ex: `django`, `htmx.org`, `tailwindcss`)
2. Buscar docs: `get-library-docs` com o ID + tópico específico

**Libs prioritárias do projeto**:

- `django` — ORM, Forms, Views, Auth, Middleware
- `htmx.org` — atributos, eventos, extensões
- `tailwindcss` — classes, @theme, @utility (especialmente v4 CSS-first)
- `mercadopago` — SDK de pagamentos, webhooks

### Playwright — Validação visual no browser

Controla um browser real para validar renderização, HTMX e UX. Configurado em `.vscode/mcp.json`.

**Quando usar**: após implementar/alterar templates, views ou frontend.  
**Quando NÃO usar**: tarefas puramente backend (models, services, migrations).

**Workflow**:

1. Garantir que `python manage.py runserver` está rodando (background)
2. Navegar para a URL da feature (`browser_navigate`)
3. Capturar snapshot acessível (`browser_snapshot`) — preferir sobre screenshot
4. Validar: elementos presentes, paleta `solar-*`, responsividade, HTMX
5. Se necessário, interagir (click, type) para testar fluxos dinâmicos
6. Corrigir problemas encontrados antes de finalizar

**Agentes com acesso**:

- `implementer`, `django-dev`, `frontend-dev` — acesso completo (navegar, clicar, digitar, screenshot)
- `reviewer` — acesso read-only (apenas navegar e snapshot)
- `planner`, `security-auditor` — sem acesso (não precisam de browser)

---

## Ficheiros de contexto

Documentação de referência que agentes devem consultar:

| Ficheiro                           | Propósito                                                    |
| ---------------------------------- | ------------------------------------------------------------ |
| [ARCHITECTURE.md](ARCHITECTURE.md) | Arquitetura técnica, convenções de código, estrutura de apps |
| [PRODUCT.md](PRODUCT.md)           | Regras de negócio, glossário, personas, gates                |
| [project-plan.md](project-plan.md) | Plano completo do ERP Solar                                  |

### Instructions (aplicadas automaticamente por tipo de arquivo)

| Arquivo                            | `applyTo`                |
| ---------------------------------- | ------------------------ |
| `python-models.instructions.md`    | `**/models.py`           |
| `python-views.instructions.md`     | `**/views.py`            |
| `django-templates.instructions.md` | `**/templates/**/*.html` |
| `python-tests.instructions.md`     | `**/test*.py`            |
| `tailwind-css.instructions.md`     | `static/css/**`          |
| `migrations.instructions.md`       | `**/migrations/**`       |

### Agentes especializados

| Agente             | Missão                                             |
| ------------------ | -------------------------------------------------- |
| `planner`          | Analisa codebase e cria planos (read-only)         |
| `implementer`      | Executa planos, cria código, valida                |
| `reviewer`         | Revisa código contra regras do projeto (read-only) |
| `django-dev`       | Dev full-stack Django + HTMX + Tailwind            |
| `frontend-dev`     | Especialista Tailwind CSS 4 + HTMX 2.x             |
| `security-auditor` | Auditoria de segurança e LGPD (read-only)          |

### Skills (receitas reutilizáveis)

| Skill                 | O que faz                               |
| --------------------- | --------------------------------------- |
| `django-app-scaffold` | Cria app Django completa com convenções |
| `htmx-partial`        | Cria par view+partial HTMX              |
| `workflow-gate`       | Implementa gate de transição de status  |
| `tailwind-component`  | Cria componente visual reutilizável     |

### Prompts (atalhos para tarefas comuns)

| Prompt            | Ação                                      |
| ----------------- | ----------------------------------------- |
| `new-app`         | Cria nova app Django completa             |
| `new-model`       | Adiciona model a app existente            |
| `review-security` | Auditoria de segurança e LGPD             |
| `run-checks`      | Roda todas as verificações de qualidade   |
| `visual-check`    | Validação visual de página com Playwright |

---

## Workflow de desenvolvimento com agentes

### Fluxo recomendado

1. **Planejamento**: Use `@planner` para analisar o que precisa ser feito. Consulta Context7 para APIs.
2. **Implementação**: Use `@django-dev` para features completas, ou `@implementer` para seguir um plano.
3. **Frontend**: Use `@frontend-dev` para melhorar templates, UX e componentes visuais.
4. **Validação visual**: Após alterar frontend, use Playwright (`browser_snapshot`) para verificar renderização.
5. **Revisão**: Use `@reviewer` para validar código contra as regras.
6. **Segurança**: Use `@security-auditor` antes de deploys ou quando tratar dados pessoais.

### Atalhos com prompts

- Nova app? → `/new-app`
- Novo model? → `/new-model`
- Verificação geral? → `/run-checks`
- Auditoria de segurança? → `/review-security`
- Validação visual? → `/visual-check`

---

## Onde começar

1. Validar que fluxo e estados estão iguais ao [project-plan](project-plan.md).
2. Implementar primeiro o caminho crítico até `ReadyToInstall` com bloqueio por entrada paga.
3. Só depois evoluir telas de instalação e pós-venda.

---

## Checklist para mudanças

- [ ] Respeita o fluxo oficial?
- [ ] Bloqueia instalação sem entrada paga confirmada?
- [ ] Mantém arquitetura templates-first + HTMX?
- [ ] Evita dependências fora do MVP sem ADR?
- [ ] Inclui impacto em segurança/LGPD quando aplicável?

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/danbio)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/danbio)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
