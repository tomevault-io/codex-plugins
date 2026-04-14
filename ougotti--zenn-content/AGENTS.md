# Copilot Instructions

## ドキュメント言語

このリポジトリでは、**日本語を主言語としてドキュメントを作成してください。**

- 記事、コメント、ドキュメントは日本語で記述します
- コードのコメントも日本語で記述します
- コミットメッセージは日本語または英語のどちらでも構いません

## Documentation Language

Please use **Japanese as the primary language** for documentation in this repository.

- Articles, comments, and documentation should be written in Japanese
- Code comments should also be written in Japanese
- Commit messages can be in either Japanese or English

## このプロジェクトでの手順メモ（2026-02-26）

- 依存関係がない状態だと `npx zenn ...` が失敗するため、最初に `npm install` を実行する
- 記事の認識確認は `npx zenn list:articles` を使う
- 記事ファイルは front matter の必須項目（`title` / `emoji` / `type` / `topics` / `published`）を埋める
- `title` が空だと保存・検証でエラーになるため、空の下書きは削除するかタイトルを入れる

## 記事公開前のチェック・校正・語調ルール（2026-02-28）

### 1) 記事ファイル／画像配置ルール

- 記事ファイル名に `draft` は使わず、内容が分かるスラッグにする（例: `ai-kakei-shisan-dashboard.md`）
- 記事本文（Markdown）は `articles/` 配下に置く
- 画像は `images/<article-slug>/` 配下にまとめる
- 記事内画像参照は相対パスで統一する（例: `../images/<article-slug>/xxx.png`）

### 2) 公開前チェック（プライバシー／不適切表現）

- 個人情報・特定につながる情報（氏名、ID、口座情報、住所、メール、電話番号）を含めない
- サービス固有名詞が個人特定の手掛かりになる場合は一般化する（例: 「家計アプリ名」「証券会社名」）
- スクリーンショットは目視で最終確認し、機微情報や固有名詞が残っていればマスクする
- セキュリティやリスクに関する表現は断定しない（例: 「絶対安全」「リスクがない」は避ける）

### 3) 校正ルール

- 地の文は「です・ます調」で統一する
- 引用文（チャットログ）・コードブロックは原文を尊重し、必要最小限のみ修正する
- 表記ゆれをなくす（例: `Chart.js`）
- 冗長な連続文は、意味を変えずに簡潔で読みやすい文へ整理する

### 4) 最終確認コマンド

- 記事認識チェック: `npx zenn list:articles`
- front matter 必須項目が埋まっていることを再確認する（`title` / `emoji` / `type` / `topics` / `published`）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/ougotti) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
