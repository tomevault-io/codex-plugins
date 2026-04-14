
# Regra de hooks (useCallback/useMemo)

- Use `useCallback`/`useMemo` somente quando houver ganho real (evitar re-renders ou custo elevado); não como padrão automático.
- Garanta dependências corretas: se o callback/memo usa um valor, ele precisa aparecer nas dependências.
- Evite criar lógica pesada no corpo do render; prefira helper/mapeamento fora do componente quando possível.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Mavimarmara)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/Mavimarmara)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
