# GitHub Copilot 指示書

目的: このリポジトリで自動化エージェント（Copilot 等）が素早く安全に作業できるよう、プロジェクト構造・ビルド手順・重要慣習を簡潔にまとめます。

クイックスタート（ローカルで確認する手順）
- 依存インストール: `bundle install`
- 開発サーバ起動: `bundle exec jekyll serve` （または `bundle exec jekyll serve --livereload`）
- 本番ビルド: `bundle exec jekyll build` → 出力は `_site/`

主要構成と役割
- `_posts/` : ブログ・レポート記事。ファイル名は `YYYY-MM-DD-*.markdown` の形式で、Jekyll のフロントマターを含む。
- `_includes/` : 部分テンプレート（例: `header.html`, `footer.html`）。ページやレイアウトから `include` される。
- `_layouts/` : ページレイアウト（例: `home.html`, `blog.html`, `page.html`）。各投稿やページはここを参照する。
- `_data/` : 構造化データ（例: `_data/schedule.yml`）。テンプレートから `site.data` 経由で参照される。
- `assets/` : CSS/JS/画像等の静的リソース。
- `_config.yml` : サイト全体の設定（プラグイン、paginate, timezone 等）。
- `Gemfile` : Jekyll / GitHub Pages 関連の gem を管理。常に `bundle exec` で実行すること。

プロジェクト固有の慣習・注意点
- Jekyll + GitHub Pages 構成。プラグインは `_config.yml` と `Gemfile` に併記されているため、バージョンや追加プラグインを変更する際は両方を確認する。
- ページ変更後は `bundle exec jekyll serve` を再起動する（`_config.yml` は自動再読み込みされない旨の注記あり）。
- ページ断片は `_includes` に置き、複数レイアウトで再利用する。例えばヘッダは `_includes/header.html` を直接編集すると全ページに反映される。
- 記事は `_posts/` に追加し、フロントマター（title, date, layout 等）を明示する。

変更・編集のヒント（具体例）
- ヘッダを小さく修正するなら: `_includes/header.html` を編集。
- トップページのテンプレートは `_layouts/home.html` を確認。
- スケジュール表示は `_data/schedule.yml` を変更し、該当レイアウトで `site.data.schedule` を参照するパターンを踏襲する。

何を期待するか（エージェントへの指示）
- 変更は最小限に留め、既存のテンプレート構造を壊さないこと。
- ビルド手順を必ず `bundle exec` を使って検証すること。
- 新しいファイルを追加する際は、対応するレイアウト/インクルードを作成して一貫性を保つこと。

参照ファイル（確認用）
- `_config.yml`, `Gemfile`, `_posts/`, `_includes/header.html`, `_layouts/home.html`, `_data/schedule.yml`

不明点や追加で含めたい情報があれば教えてください。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/coderdojo-nisshin) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
