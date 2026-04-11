# Gemini Guidelines

- never remove production docker volumes: This means never use -v when running on non-test containers:
  docker -f docker/docker-compose.yml compose down -v
- after each major code change, run linters to check if some lint errors need manual fixing:
 ruff check . --fix 
 pyright

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/KristjanHS)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/KristjanHS)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
