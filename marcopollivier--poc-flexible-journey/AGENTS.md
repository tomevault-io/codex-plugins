# Journey Control POC - Instruções para Agentes de Codificação IA

## 📚 Documentação Principal

**IMPORTANTE**: Antes de qualquer modificação no projeto, sempre consulte a documentação em `docs/`:

- **[docs/ARCHITECTURE.md](../docs/ARCHITECTURE.md)** - Arquitetura completa, componentes, decisões de design, limitações
- **[docs/JOURNEYS.md](../docs/JOURNEYS.md)** - Jornadas UNEF e UEF, steps, contratos de API, screen types
- **[docs/DEVELOPMENT.md](../docs/DEVELOPMENT.md)** - Setup, migrations, testes, troubleshooting, como adicionar jornadas

Ao fazer alterações no código, **sempre atualize a documentação correspondente** para manter a consistência.

---

## Visão Geral do Projeto

Journey BFF (Backend for Frontend) que desacopla fluxos de jornada mobile da lógica hardcoded. Controla jornadas dinâmicas através de configurações JSON com versionamento e estado persistente.

### Componentes

- **journey-bff/**: Go 1.23 + Gin + GORM + PostgreSQL
- **journey-frontend/**: Node.js 18 + Express (proxy) + Vanilla JS
- **PostgreSQL 15**: Configurações de jornadas, sessões, histórico
- **Bruno API Tests**: Testes completos para UNEF e UEF

---

## Jornadas Atuais

### UNEF (1 limite + endereço)

```
PRE_OFFER → OFFER → DUE_DATE → ADDRESS_CONFIRMATION → END
```

### UEF (2 limites + PIX)

```
PRE_OFFER → OFFER → DUE_DATE → PIX_KEY → END
```

**Limites são dinâmicos**: fornecidos no `initial_context` ao chamar `/journey/start`, não hardcoded na configuração.

---

## Padrões e Convenções

### API Contract (JourneyResponse)

```json
{
  "journey_id": "unef_v1.0.0",
  "session_id": "uuid",
  "context": { "limits": [...], "user_data": {...} },
  "navigation": {
    "current": "OFFER",
    "next": "DUE_DATE",
    "progress": { "step": 2, "total": 5 }
  },
  "actions": ["submit", "cancel"]
}
```

### Estrutura Go

- **internal/journey/engine.go**: Transições de estado, merge de initial_context
- **internal/handlers/**: HTTP handlers + DTOs (journey_id, session_id, navigation)
- **internal/models/**: GORM models com JSON fields

### Frontend

- **public/screens/**: Classes JavaScript para cada step type
- **screen-manager.js**: Mapeia step types para screen classes
- **journey-engine.js**: Client-side state management
- Sem build - serve estáticos direto

### Database

- **migrations/**: SQL executados automaticamente no primeiro startup via Docker volume mount
- Tabelas: `journeys`, `user_journey_sessions`, `journey_navigation_histories`
- Jornadas armazenadas como JSONB

---

## Comandos Essenciais

> ⚠️ **Sempre prefira os comandos do Makefile** na raiz do projeto ao invés de rodar `docker compose` diretamente.

```bash
make up       # Sobe todos os containers
make down     # Para todos os containers
make restart  # Restart sem rebuild
make rebuild  # Rebuild e restart do frontend (mudanças de UI)
make reset    # Hard reset completo: destrói tudo e recria do zero
make logs     # Exibe logs de todos os serviços
```

> O `make reset` é o comando correto para hard reset — usa `docker stop/rm` por nome de container em vez de `docker compose rm`, que não funciona com podman-compose.

---

## Regras Importantes

1. **Limites dinâmicos**: Sempre enviados no `initial_context`, nunca hardcoded
2. **Versionamento**: Usuários em jornadas antigas não são impactados por novas versões
3. **Migrations automáticas**: Executam no primeiro startup, arquivos em ordem alfabética
4. **Documentação sincronizada**: Ao alterar código, atualizar `docs/` correspondente
5. **Testes Bruno**: Usar variáveis de ambiente para session_id e journey_id
6. **Docker/Podman**: Scripts detectam automaticamente qual usar

---

## Ao Modificar Jornadas

1. Consultar [docs/JOURNEYS.md](../docs/JOURNEYS.md) para estrutura
2. Criar/atualizar screen classes se necessário
3. Adicionar migration SQL (005\_\*.sql)
4. Atualizar testes Bruno
5. Atualizar documentação
6. Rebuild: `docker compose down -v && docker compose up -d`

---

## Pontos de Atenção

- Frontend comunica via proxy Express (evita CORS)
- Session management via UUID, sem autenticação
- Context armazenado como `models.JSON` (map[string]interface{})
- Histórico completo de navegação para analytics
- Sem cache de configurações (consulta DB sempre)
- Sessões não expiram automaticamente

Ao modificar jornadas, atualizar registros do banco de dados, não arquivos JSON. Os arquivos JSON na raiz são apenas exemplos/templates.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/marcopollivier)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/marcopollivier)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
