- global state lives exclusively in `AppContext` (React context + reducer) – do not introduce other global state libraries (redux, recoil, zustand, etc.)
- interact with global state via `useAppContext()` and dispatch predefined action types in `AppContext.tsx`
- for transient ui state (modals, tooltips) use local `useState` in the component

---
> Source: [joseph-holland/op-patchstudio](https://github.com/joseph-holland/op-patchstudio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-06 -->
