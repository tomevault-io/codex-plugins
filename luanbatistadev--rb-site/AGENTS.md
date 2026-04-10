@AGENTS.md

# Code Style - Next.js Web

## Princípios

### KISS (Keep It Simple, Stupid)
- Código simples e direto
- Evitar abstrações desnecessárias
- Preferir soluções óbvias

### DRY (Don't Repeat Yourself)
- Extrair código repetido para funções/componentes reutilizáveis
- Usar constantes para valores repetidos

### YAGNI (You Aren't Gonna Need It)
- Não implementar funcionalidades "para o futuro"
- Implementar apenas o necessário agora

## React/Next.js/TypeScript

### Componentes
- Um componente por arquivo
- Componentes reutilizáveis em `components/ui/`
- Seções da página em `components/sections/`
- Layout (header, footer) em `components/layout/`
- Componentes complexos ou reutilizáveis devem ser extraídos para seus próprios arquivos
- `"use client"` apenas quando necessário (hooks, interatividade)

### State Management
- `useState` para estado simples local
- `useRef` para referências DOM e valores mutáveis
- Props drilling para 1-2 níveis, Context para estado compartilhado mais profundo
- Preferir Server Components sempre que possível

### Separação de Responsabilidades
- Pages são apenas composição de componentes — não colocar lógica complexa
- Lógica de negócio em hooks customizados ou `lib/`
- Dados estáticos em `data/`

### Nomenclatura
- Componentes: PascalCase (`Button.tsx`, `HeroSection.tsx`)
- Funções/variáveis: camelCase
- Constantes: camelCase
- Arquivos de componentes: kebab-case (`custom-cursor.tsx`)
- Types/Interfaces: PascalCase

### Imports
- Usar path alias `@/` para imports do `src/`
- Ordenar: react, next, packages externos, internos

### Estrutura de Arquivos
```
src/
├── app/[locale]/
│   ├── layout.tsx
│   ├── page.tsx
│   └── dictionaries/
├── components/
│   ├── layout/
│   ├── sections/
│   └── ui/
├── data/
├── lib/
└── proxy.ts
```

## Boas Práticas

### Geral
- **SEM comentários** - código deve ser autoexplicativo
- Nomes descritivos para variáveis, métodos e classes
- Sem emojis em código ou logs
- Funções curtas e focadas
- Early return para reduzir aninhamento

### Next.js 16
- `params` é sempre Promise — usar `await params`
- `proxy.ts` no lugar de `middleware.ts`
- `preload` no lugar de `priority` no componente Image
- Turbopack é padrão — config top-level

### Styling
- Tailwind CSS v4 com utility classes
- Design tokens via CSS custom properties em `globals.css`
- Usar classes do Tailwind, evitar inline styles (exceto gradientes complexos)
- Respeitar o design system existente

### Localização
- **NUNCA** usar textos hardcoded na UI
- Todos os textos visíveis ao usuário devem estar nos dicionários (`dictionaries/`)
- Adicionar strings em todos os idiomas: pt-BR, en

### Assets
- Imagens em `public/images/`
- SVGs como componentes ou via `<img>` / `<Image>`

## Workflow Obrigatorio

### Antes de implementar
- Ler a spec em `docs/superpowers/specs/` se existir
- Consultar design Figma local em `docs/figma/` se existir
- Confirmar com o usuario o que vai ser feito (resumo em 2-3 bullets)

### Durante
- Figma e fonte de verdade para visual (cores, espacamentos, tipografia, layout)
- Nao tomar decisoes criativas sem perguntar
- Se algo no Figma nao fizer sentido tecnico, perguntar antes de adaptar

### Depois
- `next build` deve passar
- `npx playwright test` deve passar
- Se existir referencia Figma, comparar screenshot
- Executar `code-simplifier`
- Verificar localizacao de textos (todos os idiomas)

### Regra de ouro
- Na duvida, perguntar. Nunca assumir.

## Figma Local

- Designs ficam em `docs/figma/` (screenshots + specs)
- 1 chamada MCP por secao, salvar resultado localmente
- Usuario complementa espacamentos manualmente
- Arquivos versionados no git

## Testes Playwright

- `npm test` roda todos os testes
- `npm run test:smoke` roda smoke tests
- `npm run test:visual` roda visual regression
- `npm run test:update` atualiza snapshots quando mudanca visual for intencional

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/luanbatistadev)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/luanbatistadev)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
