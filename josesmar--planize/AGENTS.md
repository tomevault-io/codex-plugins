
# Servidor local (Planize)

- **`npm run dev`** usa **`vite --mode production`**: mesmas variáveis que o build (`.env.production`, `.env.production.local`). PWA ativa como em prod.
- **`npm run dev:local`** usa modo development (`.env.development`) se precisar de env só para debug.
- Se o browser ficar preso a cache em dev: URL com **`?planize-clear-cache=1`** (limpa SW/cache uma vez) ou hard refresh / janela anónima.
- Reinicie o Vite após mudar ficheiros `.env*`.

---
> Source: [Josesmar/planize](https://github.com/Josesmar/planize) — distributed by [TomeVault](https://tomevault.io/claim/Josesmar).
<!-- tomevault:4.0:agents_md:2026-04-19 -->
