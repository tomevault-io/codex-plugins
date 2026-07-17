# ReEarth GeoSuite プラグイン開発手順

## プロジェクト概要
このリポジトリは Re:Earth 向けのプラグイン／ウィジェット群です。
- ソース: `geo_suite/src/`（TypeScript）
- マニフェスト: `plugin/reearth.yml`
- 出力: `dist/` および `release/`

## コーディング規約
- 言語: TypeScript（`tsconfig.json` の strict モード準拠）
  - 新規ファイル: 可能な限り厳密な型付けを行い、`@ts-nocheck` は使用しないこと。
  - 既存ファイル: レガシーな緩い型付けが残る可能性はあるが、新たに抑制を追加しないこと。
- ドキュメント: コメント、ドキュメント、コミットメッセージは**日本語**で記述すること。
- 型安全: `any` の使用は最小限に抑えること。
  - `reearth` グローバルの型が提供されていない場合は一時的に `any` を使って構わないが、プラグイン固有のデータ構造はインターフェースで定義すること。

## Re:Earth プラグインに関する注意点
- グローバルオブジェクト: `reearth` グローバル経由でプラグイン機能とやり取りする。
- マニフェスト連携: `plugin/reearth.yml` はエントリポイントやウィジェット構成の変更を反映させること。
- ディレクトリ構成:
  - `geo_suite/src/*.ts` → コンパイル後 `*.js` が出力される（ルートまたは `dist/`）
  - 画像等のアセットは `image/` に配置する。

## ビルドとデプロイ (Vercel)
- コマンド: `npm run build` でバンドル、アセットコピー、ZIP作成（`release/`）、および Vercel 用出力（`/vercel/output/static`）を行う。
- 設定: `vercel.json` はルーティングやCORS設定を管理する（外部からのプラグイン読み込みに重要）。
- テスト: ローカルでビルドし、Re:Earth にアップロードして表示を確認すること。

## ビルドとパッケージ化（追記）
- ソース: `geo_suite/src/` に TypeScript のソースが格納されています。
- コンパイル & パッケージ: 通常は `npm run build` でコンパイル、成果物コピー、ZIP 生成まで実行されます。
- 自動化用 Python スクリプト: 手動または CI でのビルドを簡便にするため、`scripts/build_py.py` を用意しています。存在しない場合は作成してください。
  - 役割: `prepare_default_inspector.js` を実行して `.ts` 内のデフォルトインスペクタテキストを埋め、`tsc` でコンパイルし、`dist/` に成果物を配置し、`dist/artifacts/geo_suite.zip` を作成します。
  - 使い方（例）:

```bash
# npm が利用可能な環境で
python scripts/build_py.py
# または
npm run build
```

 - 出力先:
  - 中間: `geo_suite/build/`
  - 配布: `dist/`
  - ZIP: `dist/artifacts/geo_suite.zip`（Re:Earthへアップロード可能）

 - 備考: `type: "cesium"`（Cesium World Terrain）など Cesium ION に依存する機能を有効にする場合、Re:Earth のシーン設定に Cesium ION のアクセストークンが必要です。

---
> Source: [yamamoto-ryuzo/ReEarth_GeoSuite](https://github.com/yamamoto-ryuzo/ReEarth_GeoSuite) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-17 -->
