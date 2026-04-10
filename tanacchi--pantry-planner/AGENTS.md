# pantry-planner コーディング・運用インストラクション

## 基本方針

- 本プロジェクトの実装・テスト・運用は、必ず `docs/` および各サブプロジェクト（api/ui/mcp）の `docs/` 配下ドキュメントの指示・ガイドラインに厳密に従うこと。
- 仕様・設計・命名・テスト・CI/CD・セキュリティ等、迷った場合や判断が必要な場合は必ず該当ドキュメントを参照し、指示に従うこと。
- 例外・分岐・エラーケース・型安全性・ESLint エラー解消・Arrange-Act-Assert パターン等、各種運用ルールも必ず遵守すること。

## ドキュメント参照ルール

修正内容に応じ、要求や作業内容に必要十分なドキュメントを参照し、指示に従うこと。
関連するドキュメントは必ず参照する一方で、無闇にドキュメントを参照することは避け、必要な情報のみを参照すること。

- **api/ 配下のコードを修正・実装する場合**: `api/docs/` 内の `README.md`, `guideline.md`, `technology.md` を必ず参照し、記載された設計・実装・運用方針に従うこと。
- **ui/ 配下のコードを修正・実装する場合**: `ui/docs/` 内の `README.md`, `guideline.md`, `technology.md` を必ず参照し、記載された設計・実装・運用方針に従うこと。
- **mcp/ 配下のコードを修正・実装する場合**: `mcp/docs/` 内の `README.md`, `guideline.md`, `technology.md` を必ず参照し、記載された設計・実装・運用方針に従うこと。
- **テスト実装・修正時**: `docs/test-strategy.md` を必ず熟読し、テスト設計・分岐網羅・モック運用・カバレッジ・命名・Arrange-Act-Assert パターン等の指針に従うこと。
- **API 設計・エラー処理**: `docs/api-guideline.md`, `docs/error-handling.md` を参照。
- **命名・スタイル**: `docs/naming-convention.md`, `docs/code-style.md` を参照。
- **CI/CD・環境変数**: `docs/ci-cd.md`, `docs/env-config.md` を参照。
- **DB/マイグレーション**: `docs/migration.md` を参照。
- **UI/UX**: `docs/ux-ui.md` を参照。

## 各 docs ディレクトリの概要

- `api/docs/` : API サーバの設計・実装・運用・技術方針を集約。新規参入者・レビュワーも必ず参照。
- `ui/docs/` : フロントエンド（UI）の設計・実装・運用・技術方針を集約。新規参入者・レビュワーも必ず参照。
- `mcp/docs/` : Model Context Protocol (MCP) サーバ/クライアントの設計・運用・技術方針を集約。
- `docs/` : プロジェクト全体の横断的な設計・テスト・CI/CD・命名・スタイル・API・セキュリティ等のガイドライン。

## 参照・遵守が必須となる条件例

- api/ui/mcp 配下のコード・テスト・設定を修正/追加する際は、必ず該当ディレクトリの `docs/` 内ガイドラインを参照し、指示に従うこと。
- テスト実装時は `docs/test-strategy.md` の内容を厳守すること。
- 迷った場合・判断に困った場合は、必ず関連する `docs/` 配下のドキュメントを確認し、記載内容を最優先とすること。

---

> **備考:**
> 本インストラクションおよび各種ガイドラインは随時最新化されるため、常に最新版を参照してください。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tanacchi)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/tanacchi)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
