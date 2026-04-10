# GitHub Copilot Instructions for wc-secure-pastebin

## Project Overview
This is a Vue 3 web component project for a secure pastebin application. It uses Vite for building, Buefy for UI components, Pinia for state management, and Web Crypto API for encryption.

## Coding Standards
- Use Vue 3 Composition API with `<script setup>`.
- Prefer reactive refs and computed properties from Pinia stores.
- Follow the existing file structure: components in `src/components/`, stores in `src/stores/`, utilities in `src/lib/`.
- Use async/await for asynchronous operations.
- Handle errors gracefully and provide user feedback via the injected `notify` function.

## Key Components and Patterns
- **PasswordManager.ce.vue**: Manages password generation, editing, and copying. Uses BIP39 for mnemonic generation.
- **Pasteref.ce.vue**: Main component that provides `notify` and `copyUrl` via Vue's provide/inject.
- **Store (pastebin.js)**: Pinia store for managing application state like password, encrypted text, etc.
- **Clipboard Utils**: Use `copyToClipboard` from `clipboardUtils.js` for copying to clipboard.
- **Crypto Utils**: Use functions from `cryptoUtils.js` for encryption/decryption operations.

## Notifications
- The main component `Pasteref.ce.vue` provides a `notify` function via Vue's provide/inject pattern.
- In child components, inject the `notify` function using `const notify = inject('notify')`.
- Use `notify(message, type, duration)` for user feedback instead of console.log or alerts.
- Available types: 'is-success', 'is-danger', 'is-warning', 'is-info'.
- Default duration is 2000ms if not specified.

## Encryption
- Ensure all encryption/decryption uses the password from the store.
- Handle decryption errors by setting `store.decryptError` to true.

## UI/UX
- Use Buefy components consistently.
- Provide tooltips and icons for better user experience.
- Ensure responsive design and accessibility.

## Development Workflow
- Run `npm run dev` for development server.
- Build with `npm run build`.
- Test encryption/decryption thoroughly.
- Validate that passwords are handled securely (never logged, etc.).

## Security Considerations
- Never expose passwords in logs or URLs unless explicitly intended.
- Use secure random generation for passwords.
- Ensure clipboard operations are secure and user-consented.

## File Naming
- Use `.ce.vue` for custom element components.
- Follow kebab-case for file names.

## Dependencies
- Vue 3, Buefy, Pinia, Vite, bip39, Web Crypto API.
- Avoid adding unnecessary dependencies; prefer built-in browser APIs.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/codref)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/codref)
<!-- tomevault:4.0:agents_md:2026-04-07 -->
