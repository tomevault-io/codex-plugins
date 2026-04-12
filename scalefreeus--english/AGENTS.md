# cursor Rules for frontend
- Use **TypeScript** for all new frontend code.

- Follow the **Airbnb JavaScript style guide** (our linter is configured for this).

- All React components must use **functional components and hooks** (no class components).

- Use our internal UI library components (from `@ourcompany/ui`) instead of raw HTML elements when available (e.g., use `<Button>` from the library).

- State management: use **Redux** and our existing slices (avoid using React local state for tasks).

- API calls: use the `apiClient` provided (`import { apiClient } from '@/utils/apiClient';`) rather than `fetch` directly.

- **Testing:** 

For each new module, include a simple Jest test file skeleton.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/scalefreeus)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/scalefreeus)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
