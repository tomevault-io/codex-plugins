# Instruções para GitHub Copilot

## Regras de Commit

Antes de sugerir qualquer commit, você DEVE:

1. **Rodar lint**: Execute `npm run lint` e corrija todos os erros
2. **Rodar testes**: Execute `npm test` e garanta que todos passem
3. **Não commitar se falhar**: Se lint ou testes falharem, corrija os problemas antes de sugerir o commit

## Formato de Commit

Use Conventional Commits com referência à issue:

```
tipo: descrição curta

Closes #número-da-issue
```

Exemplos:
- `feat: adicionar componente HabitCard`
- `fix: corrigir cálculo de dias`
- `refactor: reorganizar estrutura de componentes`
- `test: adicionar testes para storage`
- `docs: atualizar README`

## Workflow

1. Implementar uma issue por vez
2. Rodar `npm run audit` (lint + tests)
3. Commitar apenas se tudo passar
4. Aguardar validação do usuário antes de seguir para próxima issue

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/dwildt)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/dwildt)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
