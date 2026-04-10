# Repository overview (monorepo)
- medetect/: Python (training/export)
  - pixi を使うときは、medetect/ にcd するか、`pixi -m medetect/pyproject.toml ...` のように -m オプションを使うこと。
- mefront/: Node.js (browser frontend/inference)
  - pnpm を使うときは、mefront/ にcd するか、`pnpm -C mefront/ ...` のように -C オプションを使うこと。

# General rules
- Prefer minimal, consistent changes. Do not mix concerns between medetect/ and mefront/ unless required.
- Before suggesting commands, check each subproject's README and config files (pyproject.toml/package.json).
- Keep generated artifacts out of Git unless explicitly tracked; document how to fetch/build them.
- When changing model I/O or label map, update shared/ metadata and any consuming code.
- issueの返答やPRコメントは日本語で書くこと。それ以外は英語でよい。
- 作業の対象外のファイルについて、変更を加えないこと。スペースを削除するとか、インポートの順番を変えるとかは、変更の対象のファイルに対してのみ行うこと。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/seeker3600)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/seeker3600)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
