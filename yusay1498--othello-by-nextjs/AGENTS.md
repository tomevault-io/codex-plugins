# Othello by Next.js - AI Coding Assistant Instructions (Copilot/Claude)

## Language Preference / 言語設定

**When providing code reviews, please respond in Japanese (日本語).** This includes:
- Pull request review comments
- Code suggestions and improvements
- Explanations of issues or bugs
- Best practice recommendations

コードレビューを行う際は、日本語で応答してください。これには以下が含まれます：
- プルリクエストのレビューコメント
- コード改善の提案
- 問題やバグの説明
- ベストプラクティスの推奨事項

## 重要: 言語設定
- **PRタイトルは英語で記述してください**
- **PRディスクリプション、コード内のコメントは日本語で記述してください**
- **コミットメッセージは英語で記述してください**（先頭大文字、プレフィックス不要、50文字以内）
- エラーメッセージやログメッセージは英語でも構いませんが、説明的なコメントは日本語を使用してください

## 重要: PR作成時の制約
- **1つのPRには1つの機能のみを含めてください**
- PRの範囲が大きくなりすぎないように、機能を適切に分割してください
- レビュー負荷を軽減するため、以下の基準で機能を分割してください：
  - 1つのPRで変更するファイル数は可能な限り少なく保つ
  - 独立して動作・テスト可能な単位で分割する
  - 複数の機能を実装する場合は、それぞれ別のPRとして作成する
- 例:
  - ✅ 良い例: 「ボードコンポーネントの実装」
  - ✅ 良い例: 「合法手判定ロジックの実装」
  - ❌ 悪い例: 「ボードコンポーネント、ゲームロジック、AI機能の実装」（複数機能を含む）

## プロジェクト概要
このプロジェクトは、Next.js 16、React 19、TypeScript、Tailwind CSSを使用したオセロ（リバーシ）ゲームの実装です。

## コードスタイルと規約

### TypeScript
- アプリケーションコード（コンポーネント、ロジック、ユーティリティなど）はTypeScriptで記述してください
- 設定ファイルやビルドスクリプトなど、JavaScriptが必要な場合は適切に判断して使用してください
- 可能な限り型推論を優先しますが、関数のパラメータと戻り値には明示的な型を追加してください
- オブジェクトの形状にはinterface、ユニオン/プリミティブにはtypeを使用してください
- strict モードのコンプライアンスを有効にしてください

### React & Next.js
- デフォルトでReact Server Components (RSC)を使用してください
- 'use client' ディレクティブは必要な場合のみ追加してください（フック、ブラウザAPI、イベントハンドラー）
- Next.js 16 App Routerの規約に従ってください
- `app/` ディレクトリ構造を使用してください
- フックを使用した関数コンポーネントを優先してください

### スタイリング
- スタイリングにはTailwind CSSユーティリティクラスを使用してください
- モバイルファーストのレスポンシブデザインに従ってください
- ユーティリティクラスを整理して読みやすく保ってください

### ファイル構成
- コンポーネント: `app/` または `components/` ディレクトリ配下の適切なディレクトリに作成してください
- 型定義: 別の `.types.ts` ファイルに定義するか、シンプルな場合はインラインで定義してください
- ユーティリティ: ユーティリティ関数は `lib/` または `utils/` ディレクトリに配置してください

## ゲームロジックのガイドライン

### オセロゲームのルール
- 8x8のボードで黒と白の駒を交互に配置します
- 有効な手は少なくとも1つの相手の駒を裏返す必要があります
- 両プレイヤーが有効な手を持たなくなったらゲーム終了です
- 勝者は駒の数で決定されます

### 状態管理
- ゲーム状態にはReactフック（useState、useReducer）を使用してください
- 必要に応じてグローバルなゲーム状態にはcontextを検討してください
- ゲームロジックをUIコンポーネントから分離してください

## テスト
- ゲームロジック関数のテストを記述してください
- エッジケース（ボード境界、有効な手がない、ゲーム終了条件）をテストしてください
- リポジトリ内の既存のテストパターンに従ってください

## パフォーマンス
- 高コストな計算はメモ化してください
- 純粋なコンポーネントにはReact.memoを使用してください
- ゲーム状態の更新時の再レンダリングを最適化してください

## コミットメッセージ
- 先頭大文字の英語で記述してください
- プレフィックス（feat:, fix: など）は不要です
- 変更内容を簡潔に記述してください
- ピリオド、カンマ、接続詞はできるだけ避けてください
- 1文でかつ50文字以内に収めてください

例:
- `Add initial board layout for Othello`
- `Fix mobile layout issue`
- `Update ESLint configuration`

## PRタイトルとディスクリプション

### 重要: PR更新時の注意事項
- **PRのコメントに対応する際、既存のPRタイトルとディスクリプションを変更しないでください**
- PRタイトルとディスクリプションは、PRを最初に作成した時に設定されたものを保持してください
- コード変更のみを行い、PRのメタデータ（タイトル、ディスクリプション）は変更しないでください
- `report_progress` ツールを使用する際は、既存のPRディスクリプションを保持してください

### 新規PR作成時のガイドライン
- **PRタイトルは英語で記述してください**
  - 簡潔で分かりやすいタイトルにしてください
  - **具体的で説明的なタイトルを使用してください**（"Addressing PR comments"のような汎用的なタイトルは避けてください）
  - PRの実際の変更内容を反映したタイトルにしてください
  - 例: `Add Japanese language configuration for Claude Code`
  - 例: `Fix TypeScript usage guidelines to be more flexible`
  - 例: `Update PR title format to use English`
- **PRディスクリプションは日本語で記述してください**
- 以下のセクションを含めてください:
  - **動機**: この変更が必要になった背景
  - **提案された変更**: 変更内容の概要と実現方法
  - **拒否された代替案**: 採用しなかった他の方法とその理由（該当する場合）
  - **関連課題**: 関連するissueへのリンク
  - **チェックリスト**: セルフレビュー、テスト完了などの確認項目

## ベストプラクティス
- 関数は小さく、焦点を絞ったものにしてください
- 再利用可能なロジックはカスタムフックに抽出してください
- 複雑なゲームロジックには日本語でコメントを追加してください
- アクセシビリティを確保してください（ARIAラベル、キーボードナビゲーション）
- エラー状態を適切に処理してください

## コード内コメントの例

良い例（日本語コメント）:
```typescript
// ボード上の有効な手をすべて取得
function getValidMoves(board: Board, player: Player): Position[] {
  // 空のマスをすべて確認
  const emptyPositions = getEmptyPositions(board);

  // 各空マスについて、駒を裏返せるか確認
  return emptyPositions.filter(pos => canFlipPieces(board, pos, player));
}
```

避けるべき例（英語コメント）:
```typescript
// Get all valid moves on the board
function getValidMoves(board: Board, player: Player): Position[] {
  // Check all empty positions
  const emptyPositions = getEmptyPositions(board);

  // For each empty position, check if we can flip pieces
  return emptyPositions.filter(pos => canFlipPieces(board, pos, player));
}
```

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/yusay1498) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
