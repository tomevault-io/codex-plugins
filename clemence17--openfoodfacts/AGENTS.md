- Projet: cache local OpenFoodFacts + app Streamlit
- Objectif: maintenir un cache local (SQLite) mis à jour régulièrement, puis afficher des métriques/calculs via Streamlit.
- Contraintes:
  - Éviter de télécharger des dumps complets par défaut (trop volumineux) ; préférer une synchronisation "recent" et paramétrable.
  - Stockage local: SQLite dans `data/off_cache.sqlite`.
  - Les scripts doivent fonctionner sous Windows (PowerShell) et être planifiables via Task Scheduler.

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/clemence17) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
