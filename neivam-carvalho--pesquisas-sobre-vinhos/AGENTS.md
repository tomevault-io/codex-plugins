<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->
<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Pesquisa sobre Vinhos & Espumantes - Configuração do Projeto

## Projeto Completo ✅

Este projeto está completamente configurado e funcional. É uma aplicação de pesquisa sobre vinhos e espumantes com design inspirado no Typeform.

### Tecnologias Utilizadas
- **Frontend**: Next.js 15, React, TypeScript
- **Styling**: Tailwind CSS
- **Animações**: Framer Motion
- **Ícones**: Lucide React
- **Database**: PostgreSQL (configurado para Railway)
- **ORM**: Prisma

### Estrutura da Pesquisa
5 seções principais:
1. **Perfil Demográfico** - idade, gênero, estado civil, etc.
2. **Hábitos de Consumo** - frequência, tipo preferido, faixa de preço
3. **Preferências** - variedades, origens, canais de compra
4. **Novidades** - eventos, vinhos em lata, vinhos naturais
5. **Contato** - dados para recomendações personalizadas

### URLs
- **Pesquisa**: `/` (página principal)
- **Analytics**: `/analytics` (dashboard com estatísticas)

### Próximos Passos para Deploy
1. **Railway**: Configurar PostgreSQL e copiar DATABASE_URL
2. **Vercel**: Conectar repositório e configurar variáveis de ambiente
3. **Migração**: Executar `npx prisma migrate deploy`

---
> Source: [neivam-carvalho/pesquisas_sobre_vinhos](https://github.com/neivam-carvalho/pesquisas_sobre_vinhos) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-09-12 -->
