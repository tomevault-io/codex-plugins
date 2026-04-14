# 開発手順書

**必ずこの手順書を最初に従い、ここに記載されていない予期しない情報に遭遇した場合のみ検索やbashコマンドにフォールバックしてください。**

## 🇯🇵 言語とコミュニケーション規則

**重要: すべてのユーザー向けコミュニケーションは日本語で行うこと**

### PR（プルリクエスト）作成時の言語規則
- **PRタイトル**: 必ず日本語で記載する
- **PR概要・説明**: 必ず日本語で記載する
- **コミットメッセージ**: 日本語で記載する
- **コメントやレビュー**: 必ず日本語でユーザーとコミュニケーションを取る
- **progress報告**: 日本語で進捗を報告する

### 例外
- ブランチ名、ファイル名、関数名など技術的な識別子は英語を使用
- コード内のコメントも日本語を使用
- 技術ドキュメント内の専門用語は英語併記可

このプロジェクトは日本語を主言語とするため、ユーザーとの全てのやり取りは日本語で行ってください。

## 効率的な作業方法

### リポジトリのブートストラップ、ビルド、テスト
```bash
pnpm install
pnpm format
pnpm build
```

### アプリケーションの実行
```bash
pnpm dev
pnpm start
```

## 開発アーキテクチャ

### 技術スタック
- **フロントエンド**: React + React Router
- **スタイリング**: Tailwind CSS + shadcn/ui コンポーネント
- **コード品質**: Biome (lint + format)

### コミット前検証
変更をコミットする前に必ず実行:
```bash
pnpm typecheck
pnpm format
```

## 開発ルール

- **編集禁止** `app/shadcn/components/ui` - これらは自動生成されたshadcn/uiコンポーネントです
- TypeScriptを厳密に使用 - `any`型を避ける
- サーバーサイドロジックにはRemixのloader/actionパターンに従う
- ErrorBoundaryコンポーネントで適切なエラーハンドリングを実装する

## 一般的な開発タスク

### コードスタイルガイドライン
- 関数型Reactコンポーネントを使用（クラスコンポーネント禁止）

## トラブルシューティング

### よくある問題と解決策
- **ビルド失敗**: まず`pnpm typecheck`を実行してTypeScriptエラーを特定

### CI/CD情報
- パフォーマンス向上のためビルドアーティファクトをキャッシュ

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/kjfsm) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
