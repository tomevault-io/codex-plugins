# Copilot Instructions for dstv-angular

## Visão Geral
Este projeto é uma aplicação Angular (CLI v13) chamada "Sakai", estruturada para múltiplos módulos e componentes, com foco em integração de serviços, autenticação e deploy em Firebase e GitHub Pages.

## Estrutura Principal
- Código-fonte: `src/app/`
  - Componentes: `components/` (ex: auth, dashboard, pages)
  - Serviços: `service/` (ex: auth-service.service.ts, customer.service.ts)
  - API e interfaces: `dstv/api/` (ex: customer.ts, dstvInterfaces.ts)
  - Core: helpers, guards, tradução
  - Layout: componentes de navegação, sidebar, topbar, menu
- Assets: `src/assets/` (imagens, dados mock, estilos)
- Configuração de ambiente: `src/environments/`
- Configuração Angular: `angular.json`, `tsconfig*.json`

## Fluxos de Desenvolvimento
- **Servidor de desenvolvimento:**
  ```pwsh
  ng serve
  ```
- **Build de produção:**
  ```pwsh
  ng build --configuration production --aot
  ```
- **Testes unitários:**
  ```pwsh
  ng test
  ```
- **Deploy Firebase:**
  ```pwsh
  firebase deploy
  ```
- **Deploy GitHub Pages:**
  ```pwsh
  ng build --configuration production --aot --base-href="./"
  ngh --dir=dist/dstv-angular/browser
  ```

## Convenções e Padrões
- Serviços seguem padrão Angular Injectable e ficam em `service/`.
- Interfaces e tipos de dados estão em `dstv/api/`.
- Componentes de layout e navegação estão em `layout/`.
- Traduções e helpers em `core/`.
- Dados mock para testes e desenvolvimento em `assets/dstv/data/`.
- Configurações de ambiente separadas para produção e desenvolvimento.

## Integrações e Dependências
- **Firebase:** Configuração em `firebase.json` e serviços em `servidor-fire.service.ts`.
- **GitHub Pages:** Build customizado para publicação estática.
- **Docker:** Arquivos de configuração para DEV e PRD em `docker for DEV/` e `docker for PRD/`.

## Exemplos de Padrões
- Serviço de autenticação: `auth-service.service.ts` (injeção, métodos de login/logout)
- Guardas de rota: `guards.guard.ts` (proteção de rotas)
- Interface de cliente: `customer.ts`, `dstvInterfaces.ts`
- Layout modular: `app.layout.component.ts`, `app.menu.component.ts`, `app.sidebar.component.ts`

## Recomendações para Agentes
- Priorize reutilização de serviços e componentes existentes.
- Siga a estrutura de módulos e separação de responsabilidades.
- Utilize dados mock de `assets/` para testes e protótipos.
- Consulte arquivos de configuração para integração e deploy.
- Mantenha consistência nos nomes e padrões de diretórios.

---

Consulte este documento ao gerar código ou sugerir mudanças. Atualize conforme novas convenções surgirem.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/adcdenis)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/adcdenis)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
