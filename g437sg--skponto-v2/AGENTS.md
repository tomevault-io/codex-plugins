<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# SKPONTO - Sistema de Controle de Ponto

Este é um sistema completo de controle de ponto desenvolvido em Flask com as seguintes características:

## Estrutura do Projeto
- **Flask** como framework web principal
- **SQLAlchemy** para ORM e gerenciamento de banco de dados
- **Flask-Login** para autenticação e sessões
- **Flask-WTF** para formulários e validação
- **Bootstrap 5** para interface responsiva
- **SQLite** como banco de dados (produção pode usar PostgreSQL)

## Funcionalidades Principais
1. Sistema de autenticação multi-nível (Admin, Trabalhador, Estagiário)
2. Controle de ponto com entrada/saída automatizada
3. Gestão de usuários e permissões
4. Sistema de atestados médicos com upload
5. Dashboards interativos com gráficos
6. Relatórios em PDF e Excel
7. Sistema de notificações em tempo real
8. Backup automático local integrado
9. Logs de segurança e auditoria

## Padrões de Desenvolvimento
- Seguir padrão MVC (Models, Views, Controllers)
- Usar blueprints para organização modular
- Implementar decorators para controle de acesso
- Validação robusta de dados e sanitização
- Tratamento de erros centralizado
- Cache inteligente para performance
- Compressão GZIP automática

## Segurança
- Senhas hasheadas com bcrypt
- CSRF protection
- Rate limiting
- Validação de uploads
- Logs de auditoria
- Sessões seguras com cookies HTTPOnly

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/G437SG)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/G437SG)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
