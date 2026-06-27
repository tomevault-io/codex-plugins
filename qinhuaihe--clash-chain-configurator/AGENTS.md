<!-- Copilot instructions for the Clash Chain Configurator repository -->

# Quick Context
- Purpose: Client-side Next.js app that builds Clash (yaml) configs from user-provided proxy providers and final proxy nodes; no server-side data storage.
- Router: Next 13+ App Router under `src/app` (UI lives in `src/app/app.tsx`).

# Big-picture Architecture
- UI: React + Next App Router. Main UI is `src/app/app.tsx` which wires dialogs and lists to a single `ConfigConfigurator` instance.
- Config generation: `src/app/clash/configurator.ts` reads `src/app/clash/baseConfig.yaml` and combines `proxy-providers`, `proxies` and `proxy-groups` to produce final YAML via `js-yaml`.
- Data surface: two in-browser state sets persisted to `localStorage` using keys `clash-chain-providers` and `clash-chain-proxy-nodes` (see `src/app/app.tsx`).

# Key Patterns & Conventions (project-specific)
- Provider types: `http` or `inline`. `http` expects a `url` + `interval`; `inline` accepts YAML payloads. Validation uses `zod` in `src/components/ProviderDialog.tsx`.
- Node import: Supported link formats are `vmess://`, `vless://`, `trojan://`, `ss://`, `hysteria2://` (see `src/components/ImportProxyNodesDialog.tsx`). QR-code images pasted on the import dialog are decoded client-side with `jsqr`.
- Automatic decoding: Base64 payloads and pasted node links will be auto-decoded/converted to YAML in the provider dialog (see `handlePayloadPaste` in `src/components/ProviderDialog.tsx`).
- Names: When importing nodes, the app deduplicates names by appending `-1`, `-2`, ... to avoid collisions (`ImportProxyNodesDialog` logic).

# Integration Points & Dependencies
- `js-yaml` is used to parse/serialize node YAML and provider payloads (see `src/app/clash/configurator.ts`).
- `yaml-loader` is configured as a devDependency to allow importing YAML (`baseConfig.yaml` is imported directly).
- UI primitives: Radix UI + Tailwind + custom components under `src/components/ui/*`. Toasts use `sonner`.

# Developer Workflows & Commands
- Run locally: `npm run dev` (Next dev server). Build: `npm run build` and `npm run start` for production.
- Lint: `npm run lint`.
- There are no unit tests in the repo; run the app and exercise dialogs manually when working on parsing/UX changes.

# Where To Make Common Changes (examples)
- Change default Clash template: edit [src/app/clash/baseConfig.yaml](src/app/clash/baseConfig.yaml#L1-L20).
- Change how YAML is composed: edit [src/app/clash/configurator.ts](src/app/clash/configurator.ts#L1-L120).
- Update node parsing (vmess/vless/ss/etc): see [src/components/ProviderDialog.tsx](src/components/ProviderDialog.tsx#L1-L120) and [src/components/ImportProxyNodesDialog.tsx](src/components/ImportProxyNodesDialog.tsx#L1-L120).
- Add or modify form validation: search for `zod` usages (e.g. provider schema in `ProviderDialog.tsx`).

# Guidance For AI Agents
- Be conservative: This is a purely client-side tool; never add server-side calls that would transmit user-provided config without explicit PR discussion.
- Keep UX parity: Prefers editing or extending existing dialogs and validation rather than introducing new global state stores; follow the `localStorage` patterns already in `src/app/app.tsx`.
- Validate inputs: When adding parsing logic for new link formats, include robust try/catch and user-facing error messages as in existing parsing helpers.
- Reference examples: When suggesting code edits, link to the exact file and a small code excerpt (use the files above as canonical examples).

# After Edits
- Smoke-test locally with `npm run dev` and exercise: add provider (http/inline), paste Base64/links, import nodes via text and QR paste, then copy/download generated YAML.

If anything here looks incomplete or you want more specific rules (naming, commit message style, or test suggestions), tell me which areas to expand.

---
> Source: [qinhuaihe/clash-chain-configurator](https://github.com/qinhuaihe/clash-chain-configurator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-06-26 -->
