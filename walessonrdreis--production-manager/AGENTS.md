# Production Manager — Regras de Respostas

## 🗣️ Comportamento do Agente

### Modo Diálogo (sem código)

- **"não code"** ou **"não implementar"**: responda **apenas via chat**, sem gerar código.
- **"documentar"** ou **"documente"**: responda via chat **e** crie o arquivo `.md` explicando o que foi criado.
- **"crie os passos"** ou **"crie os passos de implementação"**: responda via chat listando os passos, sem codificar.
- **"Vamos conversar"** ou frases semelhantes: responda **apenas via chat**, sem codificar nada. Ajude a construir ideias, melhorias e validações seguindo padrões de projetos e boas práticas. Só code quando for dito **"pode codar"**.
- Enquanto **não** for dito **"pode codar"**, não gere nenhum código — mantenha o diálogo de construção de ideias.
- Quando encontrar um bug no sistema, descreva o problema e sugira soluções, mas **não** implemente correções até que seja dito **"pode codar"**.

### Modo Codificação

- **"pode codar"**: autorização para gerar código. Siga os passos definidos na conversa, **um passo por vez**.

### 🧹 Cleanup pós-teste

- Após testar alterações (curl, refresh, etc.), **sempre mate o servidor de dev** imediatamente.
- Não mantenha servidores rodando entre interações — terminou os testes, encerre o processo.
- Use `Get-Process -Id (Get-NetTCPConnection -LocalPort 3333).OwningProcess | Stop-Process -Force` no Windows ou `taskkill /F /PID <pid>`.
- Verificar portas ocupadas com `netstat -ano | findstr :PORTA` antes de iniciar novo servidor.

---

## 📂 Diretórios Base para Contexto

Sempre busque informações nestes diretórios ao analisar o projeto:

```
apps/api/prisma
apps/api/src
apps/api/src/@types
apps/api/src/bootstrap
apps/api/src/config
apps/api/src/shared
apps/api/src/modules/integration
```

### Módulos de Integração

```
apps/api/src/modules/integration
```

### Módulo de Referência (Modelo)

```
apps/api/src/modules/integration/product-structure
```

---

## 📚 Documentação de Referência

Consulte os documentos em `docs/` para regras detalhadas:

- `docs/MODULE_TEMPLATE.md` — Template canônico para novos módulos
- `docs/MODULE_QUALITY_GATE.md` — Checklist canônico (119 itens) para avaliar maturidade de módulos
- `docs/PROJECT_MANUAL.md` — Manual completo com padrões de nomenclatura e arquitetura
- `docs/DOMAIN_NAMING_GUIDE.md` — Nomenclatura de domínio
- `docs/DECISIONS.md` — ADRs (Architectural Decision Records)
- `docs/ARCHITECTURE_GUIDE.md` — Guia de arquitetura técnica

## 🔧 Comandos do Projeto

```bash
# Desenvolvimento
pnpm --filter @production-manager/api dev

# Gerar Prisma
pnpm --filter @production-manager/api prisma:generate

# Studio Prisma
pnpm --filter @production-manager/api db:studio
```

## 🏗️ Arquitetura Base

```
API 1 (Fastify) → Integração com Omie (espelho local)
API 2            → Domínio/UX (consome read-models da API 1)
Frontend         → Interface do usuário (chama apenas API 2)
```

**Regras de Ouro:**
- Frontend nunca chama Omie ou API 1 diretamente
- API 2 nunca chama Omie
- Apenas API 1 fala com Omie via gateways (real/fake)
- Read-models não executam efeitos colaterais
- Comandos de integração sempre usam `externalRequestId` (idempotência)
- Comandos retornam 202 Accepted (eventual-consistente)


## 🔴 REGRA ZERO — PROIBIÇÃO ABSOLUTA

```
❌ Nunca usar `prisma db push`, exceto se o usuário pedir explicitamente.

❌ Nunca ignorar erro de migration.

❌ Nunca continuar execução após erro de migration.
```

***

## 🔴 1. TODA MUDANÇA NO BANCO USA MIGRATIONS

```
✅ Sempre que houver mudança no schema.prisma:
→ executar `npx prisma migrate dev --name <descritivo>`

✅ Nunca alterar banco manualmente fora de migration.
```

***

## 🔴 2. NUNCA ALTERAR MIGRATIONS EXISTENTES

```
❌ Não editar migrations antigas
❌ Não reordenar migrations

✅ Sempre criar nova migration incremental
```

***

## 🔴 3. UMA RESPONSABILIDADE POR MIGRATION

```
✅ Cada migration deve ter UMA mudança clara

Exemplos:
- add_production_order_indexes
- add_command_types_retry_failed
- add_priority_column
```

***

## 🔴 4. ENUMS — REGRA CRÍTICA

```
✅ Pode adicionar novos valores em enums

❌ Nunca remover valor de enum
❌ Nunca renomear valor de enum
```

***

## 🔴 5. DRIFT (INCONSISTÊNCIA) — REGRA OBRIGATÓRIA

Antes de rodar migrations:

```
1. executar: npx prisma migrate status

Se houver inconsistência:

❌ NÃO rodar migrate dev
❌ NÃO gerar nova migration

✅ Parar execução
✅ Informar que banco está fora de sync
```

***

## 🔴 6. NUNCA IGNORAR SHADOW DATABASE ERROR

```
❌ Não ignorar erro de shadow database

Se aparecer erro:

✅ parar execução
✅ não tentar forçar aplicação
```

***

## 🔴 7. ÍNDICES DEVEM ESTAR NO SCHEMA

```
✅ Todo índice deve existir no schema.prisma

❌ Não criar índice manual via SQL

Motivo:
Prisma pode dropar índices que não conhece
```

***

## 🔴 8. SEM SQL MANUAL FORA DE MIGRATION

```
❌ Proibido usar:
- CREATE TABLE
- ALTER TABLE
- DROP INDEX
fora de migrations

✅ SQL só dentro de migration (se necessário)
```

***

## 🔴 9. SEMPRE VALIDAR MIGRATION GERADA

Após criar migration:

```
✅ Ler arquivo migration.sql

Se houver:
- DROP inesperado
- ALTER destrutivo

❌ NÃO aplicar
```

***

## 🔴 10. CONSISTÊNCIA ENTRE AMBIENTES

```
✅ Toda mudança deve funcionar em:
- dev
- staging
- production

✅ Sempre reproduzível via migrations
```

***

# 🤖 REGRAS DE EXECUÇÃO DO AGENTE

***

## 🧠 REGRA A — DECISÃO DE BANCO

Se a tarefa envolve banco:

```
→ usar schema.prisma
→ gerar migration
→ nunca usar db push
```

***

## 🧠 REGRA B — FLUXO OBRIGATÓRIO

```
1. alterar schema.prisma
2. executar migrate dev
3. revisar migration.sql
4. validar status
```

***

## 🧠 REGRA C — PROIBIÇÃO DE ATALHO

```
❌ não simplificar usando db push
❌ não criar SQL manual "rápido"
❌ não ignorar erro
```

***

## 🧠 REGRA D — EVOLUÇÃO CONTÍNUA

```
✅ evoluir schema existente

❌ não recriar tabela sem necessidade
❌ não apagar estrutura existente
```

***

# ⚠️ REGRAS DE SEGURANÇA

***

## 🔴 PROIBIDO

```
- Rodar migration com erro
- Ignorar warning crítico
- Aplicar migration destrutiva sem validar
```

***

## ✅ OBRIGATÓRIO

```
- Garantir que migrate status = OK
- Garantir que schema reflete o banco
- Garantir que mudanças são reprodutíveis
```

***

# ✅ TEMPLATE DE VALIDAÇÃO (ANTES DE EXECUTAR)

O agente DEVE validar:

```
[ ] Alteração está no schema.prisma?
[ ] Migration foi criada corretamente?
[ ] migration.sql foi revisado?
[ ] migrate status está OK?
[ ] Não foi usado db push?
```

Se qualquer resposta = NÃO:

```
→ parar execução
```

***

## 🔧 REGRA EXCEPCIONAL — RECOVERY MODE

As regras podem ser temporariamente violadas APENAS quando:

- o histórico de migrations está corrompido
- o schema está inconsistente com o banco

Nestes casos:
✅ permitido:
- ajuste manual em migrations
- execução de SQL corretivo

❌ mas obrigatório:
- restaurar o estado consistente
- voltar a seguir todas as regras imediatamente após


# 🎯 PRINCÍPIO FINAL

```
Consistência e reprodutibilidade são mais importantes que velocidade.
```

---
> Source: [Walessonrdreis/production_manager](https://github.com/Walessonrdreis/production_manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-21 -->
