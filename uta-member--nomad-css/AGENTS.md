# nomad-css Library - Copilot Instructions

## プロジェクト概要

nomad-cssは、フレームワーク非依存のプレーンなCSSライブラリです。以下の要件を満たすために開発されています：

- **マルチフレームワーク対応**: Node.js、ASP.NET Core、PHPなど様々なフレームワークで利用可能
- **低依存性**: TailwindやBootstrapのようなライブラリへの高い依存を避ける
- **レガシーブラウザ対応**: 古いブラウザでも動作する（OKLCHなどの新技術を避ける）
- **テーマ切り替え可能**: 複数のテーマを切り替えられる設計
- **カスタマイズの制限**: テーマ確定後は大きなカスタマイズを制限

## コンポーネント実装の方針

このプロジェクトは特定のテーマに沿った実装を行うのではなく、**各テーマの実装に必要なプロパティ（CSS変数）を準備すること**が目的です。

### 基本原則

- **構造的な機能に注力**: コンポーネントの構造やレイアウトに関する機能を実装する
- **汎用的なバリエーション**: filled, tonal, outlined, text などの一般的なスタイルパターンを提供する
- **セマンティックカラーはテーマ層の責務**: nomad-css-ui層でprimary, secondary等のセマンティックバリエーションは定義しない
- **拡張ポイントの提供**: テーマ層でカスタマイズできるようCSS変数で制御可能にする

### コンポーネントの実装例

#### ボタン

```html
<!-- 構造的なバリエーション（nomad-css-ui で定義） -->
<button class="button filled">Filled</button>
<button class="button tonal">Tonal</button>
<button class="button outlined">Outlined</button>
<button class="button text">Text</button>

<!-- アイコン付きボタン -->
<button class="button"><span class="icon">★</span> With Icon</button>
<button class="button icon-only"><span class="icon">★</span></button>

<!-- セマンティックカラー（テーマ層で定義） -->
<button class="button primary">Primary</button>
```

### 実装すべき内容

| コンポーネント | nomad-css-ui層で実装する内容                               | テーマ層で実装する内容           |
| -------------- | ---------------------------------------------------------- | -------------------------------- |
| Button         | filled, tonal, outlined, text, アイコン配置, サイズ        | セマンティックカラー, 角丸, 影   |
| Input          | サイズ, 状態（disabled, readonly）, バリエーション変数定義 | ボーダースタイル, フォーカス効果 |
| Card           | レイアウト構造, elevation用CSS変数                         | シャドウ, ボーダースタイル       |
| Table          | 構造, striped, bordered, hover                             | カラーバリエーション             |

## アーキテクチャ

### 階層構造

```
┌─────────────────────────────────────┐
│           テーマ層                   │  src/themes/<theme-name>/
│   (fluent2, material3, etc.)        │  CSS変数を上書きして外観を定義
├─────────────────────────────────────┤
│       nomad-css-ui 層               │  src/nomad-css-ui/
│   (button, form-control, table...)  │  プレーンなコンポーネント定義
├─────────────────────────────────────┤
│       ユーティリティ層               │  src/utilities/
│   (palette, border, fonts...)       │  テーマ非依存のユーティリティ
└─────────────────────────────────────┘
```

### ディレクトリ構造

```
src/
├── utilities/              # ユーティリティ層（テーマ・フレームワーク非依存）
│   ├── _prefixes.scss      # CSS変数グローバルプレフィックス定義
│   ├── _palette.scss       # パレット生成 Mixin (generate-palette)
│   ├── _border.scss        # ボーダーユーティリティクラス
│   ├── _breakpoints.scss   # レスポンシブBreakpoint Mixin
│   ├── _container.scss     # レイアウトユーティリティクラス (display, flex等)
│   ├── _fonts.scss         # 見出しフォントサイズ/ウェイト変数とクラス
│   ├── _shadows.scss       # シャドウ変数とユーティリティクラス (shadow-0〜5)
│   └── _spacing.scss       # マージン・パディング・ギャップクラス (m-/p-/gap-)
│
├── nomad-css-ui/           # コンポーネント層（utilitiesに依存）
│   ├── _nomad-css-ui.scss  # エントリーポイント
│   ├── components/         # 各コンポーネント（25種）
│   │   ├── _components.scss    # コンポーネントのエントリーポイント
│   │   ├── _button.scss        # ボタン
│   │   ├── _form-control.scss  # テキスト入力
│   │   ├── _form-hint.scss     # フォームヒント
│   │   ├── _select.scss        # セレクトボックス
│   │   ├── _checkbox.scss      # チェックボックス
│   │   ├── _radio.scss         # ラジオボタン
│   │   ├── _range.scss         # レンジスライダー
│   │   ├── _progress.scss      # プログレスバー
│   │   ├── _switch.scss        # スイッチ
│   │   ├── _table.scss         # テーブル
│   │   ├── _alert.scss         # アラート
│   │   ├── _badge.scss         # バッジ
│   │   ├── _chip.scss          # チップ
│   │   ├── _spinner.scss       # スピナー
│   │   ├── _skeleton.scss      # スケルトン
│   │   ├── _accordion.scss     # アコーディオン
│   │   ├── _tooltip.scss       # ツールチップ
│   │   ├── _breadcrumb.scss    # パンくずリスト
│   │   ├── _pagination.scss    # ページネーション
│   │   ├── _card.scss          # カード
│   │   ├── _divider.scss       # 区切り線
│   │   ├── _dialog.scss        # ダイアログ
│   │   ├── _avatar.scss        # アバター
│   │   └── _list.scss          # リスト
│   └── utilities/          # nomad-css-ui 内部ユーティリティ
│       ├── _palette.scss       # パレット統合（HUE + セマンティック）
│       ├── _color-defines.scss # 共通カラー変数 (border, text, surface)
│       ├── _color-utils.scss   # テキスト/背景カラーユーティリティクラス
│       ├── _hue-colors.scss    # HSL色相環 12色定義
│       ├── _semantics.scss     # セマンティックカラー定義
│       ├── _fonts.scss         # nomad-css-ui フォント変数
│       └── _border.scss        # パレットカラーのボーダー拡張
│
└── themes/                 # テーマ層（nomad-css-ui の CSS変数を上書き）
    ├── default/            # デフォルトテーマ（最小限の上書き）
    ├── fluent2/            # Microsoft Fluent 2 風
    ├── material3/          # Google Material 3 風
    ├── material2/          # Google Material 2 風
    ├── material1/          # Google Material 1 風
    ├── ant/                # Ant Design 風
    ├── apple-hig/          # Apple HIG 風
    ├── bootstrap/          # Bootstrap 風
    ├── carbon/             # IBM Carbon 風
    ├── chakra/             # Chakra UI 風
    ├── daisyui/            # DaisyUI 風
    ├── primer/             # GitHub Primer 風
    ├── shadcn/             # shadcn/ui 風
    ├── spectrum/           # Adobe Spectrum 風
    ├── aqua/               # macOS Aqua 風
    ├── holo/               # Android Holo 風
    ├── win95/              # Windows 95 風
    ├── win7/               # Windows 7 風
    ├── cyberpunk/          # サイバーパンク風
    ├── terminal/           # ターミナル風
    ├── neumorphism/        # ニューモーフィズム風
    ├── glass/              # グラスモーフィズム風
    ├── web20/              # Web 2.0 風
    ├── skeuomorphic/       # スキューモーフィック風
    ├── linear/             # Linear.app 風
    ├── nord/               # Nord カラースキーム
    └── solarized/          # Solarized カラースキーム
    # 各テーマ共通構造:
    # ├── <theme>.scss      # エントリーポイント
    # ├── utilities/        # テーマ固有の変数定義 (_prefixes, _colors, _fonts 等)
    # └── components/       # コンポーネントの上書き
```

## カラーシステム

### CSS変数プレフィックス

| プレフィックス      | 用途                                               |
| ------------------- | -------------------------------------------------- |
| `--palette-`        | パレット基本変数（hue, saturation, color-0〜10等） |
| `--palette-color-`  | パレットカラー（0-10段階の輝度）                   |
| `--palette-accent-` | アクセントカラー（テキスト色の自動反転用）         |
| `--palette-gray-`   | グレースケール                                     |
| `--neutral-`        | ニュートラルパレット（彩度5%の青系グレー）         |
| `--font-`           | フォント関連                                       |
| `--border-color`    | ボーダーカラー                                     |
| `--text-color`      | テキストカラー                                     |
| `--surface-color`   | 背景サーフェスカラー                               |

### 明度レベル（0-10）

```scss
// src/utilities/_palette.scss
$palette-lightnesses-default: (
  0: 97%,
  // 最も明るい
  1: 93%,
  2: 85%,
  3: 75%,
  4: 65%,
  5: 50%,
  // 中間
  6: 42%,
  7: 34%,
  8: 26%,
  9: 18%,
  10: 10%, // 最も暗い
);
```

### セマンティックレベル

```scss
// デフォルト（各セマンティックカラーごとに個別に定義可）
$palette-levels-default: (
  "lighter": 2,
  "light": 3,
  "default": 5,
  "dark": 7,
  "darker": 9,
);
```

### HUE定義（HSL色相環・12色）

```scss
// src/nomad-css-ui/utilities/_hue-colors.scss
$hues: (
  "red": (
    hue: 0,
    saturation: 100%,
    accent-threshold: 50%,
  ),
  "orange": (
    hue: 30,
    saturation: 100%,
    accent-threshold: 40%,
  ),
  "yellow": (
    hue: 50,
    saturation: 100%,
    accent-threshold: 30%,
  ),
  "lime": (
    hue: 90,
    saturation: 100%,
    accent-threshold: 30%,
  ),
  "green": (
    hue: 125,
    saturation: 100%,
    accent-threshold: 30%,
  ),
  "teal": (
    hue: 170,
    saturation: 100%,
    accent-threshold: 40%,
  ),
  "cyan": (
    hue: 190,
    saturation: 100%,
    accent-threshold: 40%,
  ),
  "blue": (
    hue: 220,
    saturation: 100%,
    accent-threshold: 50%,
  ),
  "indigo": (
    hue: 240,
    saturation: 100%,
    accent-threshold: 70%,
  ),
  "violet": (
    hue: 270,
    saturation: 100%,
    accent-threshold: 70%,
  ),
  "purple": (
    hue: 290,
    saturation: 100%,
    accent-threshold: 50%,
  ),
  "pink": (
    hue: 330,
    saturation: 100%,
    accent-threshold: 55%,
  ),
);
```

### セマンティックカラー

セマンティックカラーは `src/nomad-css-ui/utilities/_semantics.scss` で定義:

- `primary` - メインアクションカラー（blue@220°, saturation: 80%）
- `secondary` - セカンダリアクションカラー（pink@330°, saturation: 80%）
- `tertiary` - 第三のカラー（teal@170°, saturation: 60%）
- `success` - 成功状態（green@125°, saturation: 60%）
- `warning` - 警告状態（yellow@50°, saturation: 80%）
- `danger` - 危険/エラー状態（red@0°, saturation: 80%）
- `info` - 情報状態（cyan@190°, saturation: 80%）
- `neutral` - ニュートラル（blue@220°, saturation: 5%）

## コンポーネント

### ボタン

```html
<!-- バリエーション -->
<button class="button">Default</button>
<button class="button filled">Filled</button>
<button class="button tonal">Tonal</button>
<button class="button outlined">Outlined</button>
<button class="button text">Text</button>

<!-- セマンティックカラー（テーマが対応している場合） -->
<button class="button primary">Primary</button>
<button class="button secondary">Secondary</button>
<button class="button danger">Danger</button>
```

### レイアウトユーティリティ

```css
/* Display */
.d-none, .d-inline, .d-inline-block, .d-block
.d-flex, .d-inline-flex, .d-grid, .d-inline-grid
.d-table, .d-table-cell, .d-table-row

/* Flex Direction */
.flex-row, .flex-row-reverse, .flex-column, .flex-column-reverse

/* Flex Wrap */
.flex-wrap, .flex-wrap-reverse, .flex-nowrap

/* Justify Content */
.justify-start, .justify-end, .justify-center
.justify-between, .justify-around, .justify-evenly

/* Align Items */
.align-items-start, .align-items-end, .align-items-center
.align-items-baseline, .align-items-stretch
```

### ボーダーユーティリティ

```css
/* Border */
.border, .border-top, .border-bottom, .border-start, .border-end

/* Border Radius */
.rounded, .rounded-circle, .rounded-pill
.rounded-top-0, .rounded-bottom-0, .rounded-start-0, .rounded-end-0
```

## テーマ

### テーマの適用方法

```html
<html data-theme="fluent2">
  <!-- または -->
  <html data-theme="material3"></html>
</html>
```

### カラースキームの適用

```html
<!-- ライトモード -->
<html data-theme="fluent2" data-color-scheme="light">
  <!-- ダークモード -->
  <html data-theme="fluent2" data-color-scheme="dark"></html>
</html>
```

### 利用可能なテーマ（27種）

| カテゴリ         | テーマ名                                                                           |
| ---------------- | ---------------------------------------------------------------------------------- |
| モダンUI         | `fluent2`, `material3`, `material2`, `material1`                                   |
| デザインシステム | `ant`, `bootstrap`, `chakra`, `carbon`, `primer`, `shadcn`, `spectrum`, `daisyui`  |
| OS風             | `apple-hig`, `win95`, `win7`, `aqua`, `holo`                                       |
| ユニーク         | `cyberpunk`, `terminal`, `neumorphism`, `glass`, `web20`, `skeuomorphic`, `linear` |
| カラー           | `nord`, `solarized`                                                                |
| デフォルト       | `default`                                                                          |

### カスタムテーマの作成

テーマは `src/nomad-css-ui/nomad-css-ui.scss` を `@use` してから、`data-theme` セレクタでCSS変数を上書きします：

```scss
@use "../../nomad-css-ui/nomad-css-ui.scss";

:root[data-theme="my-theme"] {
  // カラー
  --text-color: #1f1f1f;
  --surface-color: #f0f0f0;
  --surface-color-lighter: #ffffff;

  // ボタン
  .button {
    --button-border-radius: 4px;
    &.filled {
      --button-border-color: var(--button-bg-color);
    }
  }
}
```

## 主要Mixin

### generate-palette

パレットのCSS変数（hue, saturation, color-0〜10, accent, gray）を一括生成します。

```scss
// src/utilities/_palette.scss
@include palette.generate-palette($palette-map);

// $palette-map の構造
$example-palette: (
  hue: 220,
  saturation: 80%,
  accent-threshold: 50%,
  levels: (
    "lighter": 2,
    "light": 3,
    "default": 5,
    "dark": 7,
    "darker": 9,
  ),
  // オプション: 変数名のオーバーライド
  names: (
      palette-prefix: "my-palette-",
      palette-hue-var: "--my-palette-hue",
    ),
);
```

### media-up / media-down / media-between / media-only

レスポンシブメディアクエリを生成します。

```scss
// src/utilities/_breakpoints.scss
@include breakpoints.media-up("md") {
  /* min-width: 768px */
}
@include breakpoints.media-down("md") {
  /* max-width: 767.98px */
}
@include breakpoints.media-between("sm", "lg") {
  /* 576px〜991.98px */
}
```

**ブレークポイント**: `xs(0)`, `sm(576px)`, `md(768px)`, `lg(992px)`, `xl(1200px)`, `xxl(1400px)`

## 利用パターン

nomad-css は 3 つのパターンで利用できます（詳細: [docs/getting-started.md](../docs/getting-started.md)）。

### パターン1: ユーティリティのみ

`src/utilities/` の Sass Mixin・クラスのみを使用します。コンポーネントやテーマは不要です。

```scss
@use "src/utilities/spacing";
@use "src/utilities/border";
@use "src/utilities/container";
@use "src/utilities/fonts";
@use "src/utilities/shadows";
@use "src/utilities/palette" as palette;

// パレット生成Mixinの使用例
:root {
  @include palette.generate-palette(
    (
      hue: 220,
      saturation: 80%,
    )
  );
}
```

### パターン2: nomad-css-ui（テーマなし）

`src/nomad-css-ui/nomad-css-ui.scss` をインポートし、コンポーネントをデフォルトスタイルで使用します。
またはコンパイル済みの `dist/css/default.css` を `<link>` で読み込みます。

```scss
@use "src/nomad-css-ui/nomad-css-ui.scss";
```

```html
<!-- コンパイル済みCSSを直接使用 -->
<link rel="stylesheet" href="dist/css/default.css" />
<button class="button filled">Filled</button>
```

### パターン3: テーマを使用（推奨）

コンパイル済みのテーマCSSを読み込み、`data-theme` 属性でテーマを指定します。

```html
<link rel="stylesheet" href="dist/css/fluent2.css" />
<html data-theme="fluent2" data-color-scheme="light">
  <button class="button filled primary">Primary</button>
</html>
```

## 開発ガイドライン

### コーディング規約

1. **CSS変数のプレフィックス**: 必ずプレフィックスを使用する
2. **HSL使用**: カラーはHSL形式で定義（OKLCHは使用しない）
3. **レガシー互換性**: 古いブラウザでも動作するプロパティを使用
4. **Sassモジュール**: `@use` を使用してモジュールをインポート（`@import` 不使用）
5. **nomad-css-ui層での色直接指定禁止**: 具体的な色値（`hsl(210, 100%, 50%)`等）を直接記述しない。セマンティックカラーや共通カラー変数（`var(--border-color)`, `var(--text-color)`等）を参照する。具体的な色値はテーマ層でのみ定義する

### ファイル命名規則

- パーシャルファイル: `_filename.scss`
- エントリーポイント: `filename.scss`
- テーマファイル: `themename.scss`

### 新しいコンポーネントの追加

1. `src/nomad-css-ui/components/_newcomponent.scss` を作成
2. `src/nomad-css-ui/components/_components.scss` に `@use` を追加
3. 必要に応じて各テーマ層でCSS変数を上書き

### 新しいユーティリティの追加

1. `src/utilities/_newutility.scss` を作成
2. `src/nomad-css-ui/_nomad-css-ui.scss` に `@use` を追加

## 技術的な決定事項

- **OKLCHを避ける理由**: 古いブラウザでのサポートが不十分
- **HSLを採用した理由**: 広いブラウザサポートと直感的な色操作
- **CSS変数を活用**: テーマ切り替えをランタイムで実現
- **Sassを使用**: Mixin、変数、モジュールシステムの活用

## 開発コマンド

```bash
npm run build           # 全テーマビルド + ショーケース生成
npm run build:css       # 全テーマ(27種)のSassコンパイル
npm run build:fluent2   # 個別テーマビルド
npm run build:showcase  # ショーケースサイト生成のみ
npm run watch           # 開発サーバー起動 (ファイル変更を監視)
npm run watch:css       # Sassのみ監視
npm run format          # コードフォーマット (Prettier)
```

**ビルド成果物**: `dist/css/<theme>.css`  
**ショーケース**: `_site/`（Eleventy により生成）

## 関連ドキュメント

- [Getting Started](../docs/getting-started.md)
- [アーキテクチャ詳細](../docs/architecture.md)
- [ユーティリティリファレンス](../docs/utilities.md)
- [コンポーネントリファレンス](../docs/components.md)
- [テーマ作成ガイド](../docs/themes.md)

---
> Source: [Uta-member/nomad-css](https://github.com/Uta-member/nomad-css) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->
