
# コミットメッセージのルール

このリポジトリでは `@commitlint/config-conventional` を使用している。コミットメッセージは以下のルールに従うこと。

## 形式

```
<type>(<scope>): <subject>
```

## 制約

- **type**（必須）: `build`, `chore`, `ci`, `docs`, `feat`, `fix`, `perf`, `refactor`, `revert`, `style`, `test` のいずれか
- **scope**（任意）: 小文字。例: `darwin`, `toolchains`
- **subject**（必須）: 小文字で始める。末尾にピリオドを付けない
- **header-max-length**: 1 行 100 文字以内

## 例

```
feat(darwin): add always-on power management script
```

```
fix: correct symlink path for zsh config
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/nozomiishii)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/nozomiishii)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
