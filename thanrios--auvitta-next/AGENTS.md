# GitHub Copilot Instructions — Auvita Frontend

@version: 1.1
Last updated: March 09, 2026

## 1) Project Context

This frontend supports a multi-professional clinic. Copilot must prioritize consistency with the healthcare domain and with conventions already adopted in the codebase.

Core domain entities:
- `Patient`
- `Professional`
- `Appointment`
- `MedicalRecord`
- `Specialty`

Always prefer naming and modeling aligned with the clinical domain.

---

## 2) Stack and Versions (source: package.json)

- `next`: `16.1.6`
- `react`: `19.2.3`
- `react-dom`: `19.2.3`
- `typescript`: `^5` (com `strict: true`)
- `tailwindcss`: `^4`
- `@tailwindcss/postcss`: `^4`
- `next-intl`: `^4.8.3`
- `axios`: `^1.13.5`
- `@tanstack/react-query`: `^5.90.21`
- `@tanstack/react-query-devtools`: `^5.91.3`
- `zustand`: `^5.0.11`
- `react-hook-form`: `^7.71.1`
- `@hookform/resolvers`: `^5.2.2`
- `zod`: `^3.25.76`
- `next-themes`: `^0.4.6`
- `radix-ui`: `^1.4.3`
- `class-variance-authority`: `^0.7.1`
- `clsx`: `^2.1.1`
- `tailwind-merge`: `^3.4.0`
- `lucide-react`: `^0.563.0`
- `sonner`: `^2.0.7`
- Testing: `vitest ^4.0.18`, `@testing-library/react ^16.3.2`, `@testing-library/user-event ^14.6.1`, `@testing-library/jest-dom ^6.9.1`, `jsdom ^28.1.0`
- Linting/quality: `eslint ^9`, `eslint-config-next 16.1.6`

---

## 3) Default Language

- Code identifiers (variables, functions, types, classes, files): **English**.
- Code comments: **Brazilian Portuguese (pt-BR)**.
- UI text: use i18n (`next-intl`) and message files (`src/messages/*`), avoiding hardcoded strings.

---

## 4) Naming and Structure Conventions

- Folders and files: `kebab-case`.
- React components and component types: `PascalCase`.
- Functions, variables, and props: `camelCase`.
- Constants: `UPPER_SNAKE_CASE` only for true global constants.
- Main structure:
  - `src/app/*`: routes, layouts, pages, and App Router boundaries.
  - `src/components/ui/*`: reusable primitives (design system base).
  - `src/components/*`: feature/domain components.
  - `src/hooks/*`: reusable hooks.
  - `src/lib/*`: API layer, utilities, and infrastructure.
  - `src/types/*`: shared type contracts.

---

## 5) Import Pattern

- Prefer absolute imports with alias `@/`.
- Official alias: `@/* -> ./src/*` (defined in `tsconfig.json`).
- Import order:
  1. Bibliotecas externas
  2. Internal imports with `@/`
  3. Local relative imports (`./`, `../`) only when co-location makes sense
- Avoid long relative chains (`../../../`).

---

## 6) TypeScript Rules

- `strict` mode is mandatory and already enabled.
- Do not use `any`.
- Prefer `unknown` when needed, with explicit narrowing.
- Use `interface` for public object contracts (especially props and payloads).
- Use `type` for unions, intersections, utility types, and composite aliases.
- Type return values of exported and async functions in shared modules.
- Reuse types from `src/types/*` before creating duplicates.

---

## 7) Data and State Patterns

- Server state: use `@tanstack/react-query`.
- Global client (non-server) state: use `zustand` when needed.
- Forms: `react-hook-form` + `zod`.
- HTTP calls: centralize in `src/lib` (axios + helpers), avoid spreading across visual components.
- Handle loading/error/success explicitly in the UI.
- Do not log sensitive patient/professional data in production.

---

## 8) Accessibility (WCAG 2.1 AA) — Mandatory

- Every interactive element must be keyboard accessible.
- Always keep visible focus states (`focus-visible`).
- Ensure minimum contrast compatible with WCAG 2.1 AA.
- Inputs must have associated labels.
- Icon-only buttons must include `aria-label`.
- Error messaging cannot rely on color alone.
- Use semantic HTML (`main`, `nav`, `header`, `section`, `button`, etc.).
- Respect reduced-motion preferences when applicable.

---

## 9) Code Generation Guidelines

When generating new code, Copilot must:
1. Reuse existing components in `src/components/ui/*` before creating new ones.
2. Follow App Router with preference for Server Components.
3. Separate data logic (hooks/lib) from presentation (components).
4. Include full typing for props and API contracts.
5. Include accessibility attributes by default.

## 🌐 Localização (i18n) — Obrigatório

- **Ao gerar ou adicionar qualquer texto de UI** (labels, placeholders, validações, mensagens, tooltips), adicione também uma chave correspondente em `src/messages/` para as línguas suportadas (`pt-BR`, `en-US`) e use `next-intl` para renderizar as mensagens — **não** deixe strings hardcoded.
- Arquivo de mensagens: mantenha `src/messages/pt-BR.json` e `src/messages/en-US.json` sincronizados. Para cada nova chave, inclua traduções nas duas línguas antes do merge.
- Exemplo mínimo:

```json
// src/messages/en-US.json
{
  "auth.login.button": "Sign in"
}

// src/messages/pt-BR.json
{
  "auth.login.button": "Entrar"
}
```

- Use `useTranslations()` ou componentes `next-intl` no React para acessar as chaves.
- Nos PRs, inclua as alterações nos arquivos `src/messages/*` e certifique-se de que os testes localizados/verificações de copy não quebrem. Se houver uma ferramenta de extração automática configurada, documente o comando no PR; caso contrário, faça a atualização manualmente.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/thanrios) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
