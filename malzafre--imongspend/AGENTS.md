# ImongSpend Copilot Instructions

## Mission

Build and maintain ImongSpend as a Shopee-first spending visibility MVP.

## Product Scope

- Landing app: promote the value, explain MVP boundaries, collect early interest.
- Extension app: Chrome Manifest V3 scaffold with popup/options/background/content.
- Shared package: keep data contracts and reusable provider logic.

## MVP Boundaries

- In scope now: Shopee-first flows and data contracts.
- Out of scope now: Foodpanda/Grab integrations, production-grade ingestion, advanced dashboards.

## Engineering Rules

- Keep project structure under `apps/landing`, `apps/extension`, and `packages/shared`.
- Put cross-app models/utilities in `packages/shared`.
- Do not duplicate business types inside app folders.
- Keep extension runtime code in `apps/extension/src/background` and `apps/extension/src/content`.
- Keep popup and options UIs separate from landing UI concerns.

## Compliance Guardrail

Before adding real Shopee data extraction code, document the legal and technical basis for access (official API, export, or allowed parsing path) and keep the implementation behind an explicit MVP gate.

## Quality Gates

Run before completing major tasks:

- `npm run lint`
- `npm run typecheck`
- `npm run build`

## Coding Style

- Prefer TypeScript strictness and explicit types for contracts.
- Keep files focused and small.
- Add concise comments only when logic is non-obvious.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/malzafre)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/malzafre)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
