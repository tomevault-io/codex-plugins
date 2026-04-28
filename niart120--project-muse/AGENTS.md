# Project Muse — Copilot Instructions

## はじめに

ユーザとの対話は日本語で行うこと。

## プロジェクト概要

クリエイティブコーディングプロジェクト。Three.js (WebGPURenderer) + TypeScript + Vite による 3D ビジュアライゼーション。
テーマごとにモデルを分離し、ノードガーデンを最初のテーマとして実装している。

## 技術スタック

- **言語**: TypeScript (strict モード)
- **レンダラ**: Three.js + `WebGPURenderer` — WebGL より WebGPU を常に優先
- **GUI**: lil-gui (パラメータのランタイム操作用)
- **ビルド**: Vite (ESNext ターゲット)
- **テスト**: Vitest (jsdom 環境), カバレッジは `@vitest/coverage-v8`
- **パッケージ管理**: pnpm

## コーディング規約

- TypeScript strict mode 使用
  - class利用最小化 (関数型プログラミング推奨)
  - 型定義を厳密に行う (any, unknown, 型アサーションの多用禁止)
- ES module (`import` / `export`) のみ使用。CommonJS は禁止
- `const` を優先し、`let` は必要な場合のみ。`var` は使用禁止
- エクスポートする関数には明示的な戻り値型を付ける
- 描画ロジック (GPU / Three.js) とシミュレーションロジック (純粋な数学・データ) を分離する
- ビジュアルに影響するパラメータはすべてテーマ内の `params.ts` に集約する

## ディレクトリ構成

```
src/
├── main.ts                     # エントリポイント: テーマの起動とレンダーループ
├── core/                       # テーマ横断の共有インフラ
│   ├── renderer.ts             # WebGPURenderer / シーン / カメラ初期化
│   ├── controls.ts             # OrbitControls
│   ├── gui.ts                  # lil-gui ユーティリティ
│   └── index.ts                # core barrel export
├── models/                     # テーマ別ディレクトリ
│   └── node-garden/            # テーマ: ノードガーデン
│       ├── index.ts            # テーマのセットアップ (シーン構築, GUI, update/dispose)
│       ├── simulation.ts       # 純粋な計算ロジック (Three.js 非依存)
│       ├── params.ts           # テーマ固有パラメータ定義
│       └── mod.ts              # テーマの public re-export
└── utils/                      # 純粋ユーティリティ (数学ヘルパー等)
tests/                          # Vitest テスト (src/ のミラー構成)
public/                         # 静的アセット
```

### テーマの追加方法

1. `src/models/<テーマ名>/` ディレクトリを作成
2. 最低限 `index.ts` (setup 関数), `simulation.ts`, `params.ts` を用意
3. `src/main.ts` で import して起動

## テスト

- テストは `tests/` に配置し、`src/` のディレクトリ構成をミラーする
- `simulation.ts` と `params.ts` は必ずユニットテストを書く
- 描画コード (`main.ts`, `core/`, テーマの `index.ts`) はカバレッジ対象外

## コミットルール

- [Conventional Commits](https://www.conventionalcommits.org/) に準拠する
- フォーマット: `<type>(<scope>): <subject>`
  - `<scope>` は省略可
- 許可される type:
  - `feat` / `fix` / `docs` / `style` / `refactor` / `perf` / `test` / `build` / `ci` / `chore` / `revert`
- subject は日本語で記述・末尾句点なし

## コマンド

| タスク           | コマンド             |
| ---------------- | -------------------- |
| 開発サーバ       | `pnpm dev`           |
| ビルド           | `pnpm build`         |
| ビルドプレビュー | `pnpm preview`       |
| テスト           | `pnpm test`          |
| テスト (watch)   | `pnpm test:watch`    |
| カバレッジ       | `pnpm test:coverage` |
| リント           | `pnpm lint`          |
| フォーマット     | `pnpm format`        |
| フォーマット確認 | `pnpm format:check`  |

---
> Source: [niart120/Project_Muse](https://github.com/niart120/Project_Muse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-28 -->
