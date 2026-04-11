# Copilot Instructions

このリポジトリは **GitHub Copilot のカスタムエージェント教材**です。

## 編集方針
- 説明は **初学者向けの日本語**で簡潔に書く
- `Custom Agents` と `Skills` を混同しない
- 公式仕様にない frontmatter を断定しない
- サンプルは **最小構成から段階的に**示す

## 用語の整理
- `copilot-instructions.md`: リポジトリ全体への共通指示
- `*.prompt.md`: 単発タスク用プロンプト
- `SKILL.md`: 再利用可能な作業手順
- `*.agent.md`: 役割とツールを持つカスタムエージェント

## サンプルの場所
- 実際に使う例: `.github/agents/`
- 解説ドキュメント: `docs/`

## 品質ルール
- `description` には「いつ使うか」を具体的に書く
- `tools` は必要最小限に絞る
- 1つの agent に責務を詰め込みすぎない

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/duwenji)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/duwenji)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
