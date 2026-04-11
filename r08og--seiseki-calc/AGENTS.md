# Copilot Instructions

<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

## プロジェクト概要
このプロジェクトは「seiseki」という評定計算システムです。日本の学校制度における5段階評定（1-5）で、目標の評定を達成するために次のテストで何点取れば良いかを計算するWebアプリケーションです。React + TypeScript + Viteを使用して構築されています。

## 機能要件
- 科目別の評定計算
- 実施済みテストの点数入力・管理
- 予定テストの設定・管理
- 目標評定の設定（1-5段階）
- 目標達成に必要な点数の自動計算
- 加重平均を考慮した総合評価
- 授業態度・参加点の考慮
- 達成可能性の判定と推奨戦略の提示
- データの永続化（ローカルストレージ）

## 評定基準
- 5: 80点以上（優秀）
- 4: 65-79点（良好）
- 3: 50-64点（普通）
- 2: 40-49点（やや不良）
- 1: 39点以下（不良）

## 計算方式
- テスト80% + 平常点20%で総合評価を計算
- 平常点は0-20点の範囲で入力
- デフォルトのテスト重み：各テスト80%（1つのテストで80%を想定）
- ユーザーが重みを自由に設定可能
- 複数の予定テストがある場合は最適な点数配分を計算

## 技術スタック
- React 18
- TypeScript
- Vite
- CSS Modules または Styled Components

## コーディング規約
- TypeScriptの型安全性を最大限活用する
- 関数コンポーネントとHooksを使用する
- 適切なエラーハンドリングを実装する
- アクセシブルなUIを心がける

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/r08og)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/r08og)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
