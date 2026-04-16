<!-- Instruções personalizadas para Copilot neste workspace -->

## Projeto: Páginas Amarelas - Next.js Edition

Este é um projeto full-stack para gerenciamento de biblioteca pessoal de livros com:

- **Frontend**: Next.js 14 com React e Tailwind CSS
- **Backend**: API Routes + Prisma ORM
- **Database**: PostgreSQL
- **Autenticação**: NextAuth.js com bcryptjs

### Arquitetura

- `app/` - Páginas e API routes (App Router)
- `components/` - Componentes React reutilizáveis
- `lib/` - Utilitários (auth, validação, Prisma client)
- `prisma/` - Schema do banco de dados

### Estética Preservada

Mantemos a paleta de cores do projeto original em Tailwind:

- Primary: `rgb(26, 31, 58)` (#1a1f3a) - Azul escuro
- Secondary: `rgb(34, 211, 238)` (#22d3ee) - Ciano
- Accent: `rgb(16, 185, 129)` (#10b981) - Verde
- Gradientes: Amarelo/Laranja para CTAs

### Próximas Tarefas

1. Criar página de adição de novo livro
2. Integrar upload de imagens
3. Adicionar seed de dados de exemplo
4. Implementar testes automatizados

### Comandos Úteis

```bash
npm run dev           # Iniciar desenvolvimento
npm run build         # Build para produção
npx prisma db push   # Apply migrations
npx prisma studio   # Visualizar banco
```

### Variáveis de Ambiente Necessárias

- `DATABASE_URL` - Connection string PostgreSQL
- `NEXTAUTH_URL` - URL da aplicação (localhost:3000 em dev)
- `NEXTAUTH_SECRET` - Chave secreta para JWT

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/wikeed) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
