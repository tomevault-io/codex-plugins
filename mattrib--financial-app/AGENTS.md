# CLAUDE.md

Este arquivo fornece orientação ao Claude Code para trabalhar neste repositório.

## Visão Geral do Projeto

Aplicação de gerenciamento financeiro pessoal estilo monorepo:

| Camada | Tecnologia | Diretório |
|--------|------------|-----------|
| Frontend | React 19 + Vite + TypeScript | `src/` |
| Backend | NestJS + TypeScript | `finances-api/` |
| Database | Supabase (PostgreSQL) | - |
| Auth | Supabase Auth (JWT) | - |
| State | Zustand | `src/store/` |
| Styling | Tailwind CSS v4 | - |

## Estrutura do Projeto

```
financial_app/
├── src/                    # Frontend React (ver src/CLAUDE.md)
├── finances-api/           # Backend NestJS (ver finances-api/CLAUDE.md)
├── scripts/                # Scripts utilitários
├── supabase/               # Configuração Supabase
└── public/                 # Assets estáticos
```

## Comandos Essenciais

```powershell
# Desenvolvimento
npm run dev:all              # Frontend + Backend (recomendado)
npm run dev                  # Frontend: http://localhost:5173
cd finances-api && npm run start:dev  # Backend: http://localhost:3333

# Build & Lint
npm run build                # Frontend
npm run lint                 # Frontend
cd finances-api && npm run build      # Backend
cd finances-api && npm run lint       # Backend

# Testes
npm test                     # Frontend
cd finances-api && npm test  # Backend

# Utilitários
npm run remove-users         # Remove usuários do Supabase
```

## Variáveis de Ambiente

Arquivo `.env` na raiz (compartilhado):

```env
# Frontend (prefixo VITE_ obrigatório)
VITE_SUPABASE_URL=https://xxx.supabase.co
VITE_SUPABASE_ANON_KEY=eyJ...
VITE_API_URL=http://localhost:3333/api

# Backend
SUPABASE_URL=https://xxx.supabase.co
SUPABASE_ANON_KEY=eyJ...
SUPABASE_SERVICE_ROLE_KEY=eyJ...
PORT=3333
CORS_ORIGINS=http://localhost:5173,http://localhost:5174
```

## Fluxo de Autenticação

```
1. Login via authStore.signIn() → Supabase Auth
2. Supabase retorna session com access token (JWT)
3. authStore salva session e expõe getAccessToken()
4. API client injeta token em todas as requisições
5. Backend valida token via AuthGuard
6. Services filtram dados por user_id do JWT
```

## Fluxo de Dados (Frontend → Backend)

```
Component → Zustand Store → Service → API Client → Backend API → Supabase
```

## Adicionando Novos Endpoints

1. **Backend**: Criar DTO em `finances-api/src/modules/<domain>/dto/`
2. **Backend**: Adicionar endpoint no controller com decorators Swagger
3. **Backend**: Implementar método no service
4. **Frontend**: Atualizar types em `src/types/index.ts`
5. **Frontend**: Criar/atualizar service em `src/services/<domain>.ts`
6. **Testes**: Escrever testes para controller e service

## Convenções Importantes

- **Types**: Frontend types em `src/types/index.ts` devem espelhar os DTOs do backend
- **Testes**: Arquivos com sufixo `.spec.ts`
- **Swagger**: Documentação em `http://localhost:3333/docs`
- **Errors**: Backend usa exceções NestJS, frontend captura via `HttpError`
- **401**: Auto-logout e redirect para `/auth/login`

## Notas Técnicas

- **Vite**: Usa `rolldown-vite@7.2.5` (override no package.json)
- **Tailwind v4**: Plugin PostCSS (não CLI)
- **API Prefix**: Todas as rotas backend têm prefixo `/api`
- **Supabase CLI**: `cd finances-api && npx supabase`

## Documentação Detalhada

- Frontend: `src/CLAUDE.md`
- Backend: `finances-api/CLAUDE.md`
- **Design System: `src/DESIGN_SYSTEM.md`** (OBRIGATÓRIO para alterações de UI)

## Design System (IMPORTANTE)

**Antes de qualquer alteração no frontend, LEIA `src/DESIGN_SYSTEM.md`**

Este arquivo contém:
- Paleta de cores (Slate/Blue)
- Padrões de dark mode
- Componentes base e como usá-los
- Animações com Framer Motion
- Tipografia e espaçamento
- Checklist para novos componentes

### Resumo Rápido do Design

| Aspecto | Padrão |
|---------|--------|
| Cores Base | `slate-*` (50-950) |
| Accent | `primary-*` / `accent-*` (azul) |
| Sucesso | `emerald-*` / `green-*` |
| Erro | `rose-*` / `red-*` |
| Alerta | `amber-*` / `yellow-*` |
| Dark Mode | Classe `.dark` no `<html>` |
| Animações | Framer Motion (sutis) |
| Cards | `AnimatedCard` component |
| Gradients | **PROIBIDO** - usar cores sólidas |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/MattRib) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
