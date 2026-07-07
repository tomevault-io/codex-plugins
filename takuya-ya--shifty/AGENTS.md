# Shifty AI Coding Instructions

## このドキュメントについて

- GitHub Copilot や各種 AI ツールが本リポジトリのコンテキストを理解しやすくするためのガイドです。
- 新しい機能を実装する際はここで示す技術選定・設計方針・モジュール構成を前提にしてください。
- 不確かな点がある場合は、リポジトリのファイルを探索し、ユーザーに「こういうことですか?」と確認をするようにしてください。

## 前提条件

- 回答は必ず日本語でしてください。
- **回答前に必ず関連ファイルを実際に確認してから説明すること。**
- **推測・憶測での説明は禁止。「〜の可能性が高い」「〜と考えられます」などの表現は使わないこと。**
- **コードを確認できていない部分は「未確認」と明示し、確認方法をユーザーに提示すること。**
- 何か大きい変更を加える場合、まず何をするのか計画を立てた上で、ユーザーに「このような計画で進めようと思います。」と提案してください。この時、ユーザーから計画の修正を求められた場合は計画を修正して、再提案をしてください。

## アプリ概要

**Shifty** は、シフト作成作業を効率化し、管理者の負担を軽減するためのWebサービスです。Excelでの管理から脱却し、半月ごとのシフト登録・編集・確定といった運用を一元化します。

- **主な機能**: シフトCRUD、確定プロセス、人数集計、店休日/営業時間設定、スタッフ管理（Phase2で希望提出機能）。
- **開発フェーズ**: 現在は **Phase1 (MVP)**。管理者機能を中心に構築中。

## 技術スタック概要

### Backend (Laravel)

- **Framework**: Laravel 12 (PHP 8.2/8.5)
- **Database**: MySQL 8.4 (Docker Sail)
- **Caching/Queue**: Redis
- **Auth**: Laravel Sanctum (Cookie-based API 認証)
- **Code Quality**: PHP Strict Types, PSR-12 (Laravel Pint)
- **Testing**: PHPUnit (Unit / Feature Tests)

### Frontend (React)

- **Language**: TypeScript (strict mode ON)
- **Framework**: React 19 / Vite
- **Router**: React Router
- **State Management**:
  - **Server State**: TanStack Query (React Query)
  - **Global/UI State**: Zustand (必要に応じて)
  - **Form**: React Hook Form + Zod
- **Styling**: Tailwind CSS / shadcn/ui + Radix UI
- **Utilities**: date-fns (日付操作の標準), dnd-kit (ドラッグ&ドロップ操作)
- **API Client**: fetch API + 独自 wrapper (shared/api)
- **Testing**: Vitest + React Testing Library (Phase2以降)

## プロジェクト構成と役割

### Backend (backend/app)

```
app/
├── Http/
│   ├── Controllers/    # ルーティング入口、認可、Resource変換
│   └── Resources/      # APIレスポンス変換
├── Services/           # 業務ルール、トランザクション、横断ロジック
├── Repositories/       # DB操作(Eloquent/SQL)に特化、ビジネスロジックを持たない
└── Models/             # Eloquent モデル定義
```

### Frontend (frontend/src)

```
src/
├── features/           # 機能別モジュール (shift, auth, settings等)
│   ├── components/
│   ├── hooks/
│   ├── api/
│   └── types/
├── shared/             # 共通モジュール
│   ├── components/     # ui (shadcn等), layout
│   ├── hooks/
│   ├── utils/
│   └── api/            # APIクライアント設定
└── assets/             # 静的ファイル
```

## アーキテクチャ指針

### バックエンド (CSRパターン)

- **Controller**: 判断しない。リクエストを受取り、Serviceを呼び出し、Resourcesで返却するのみ。3行を超える処理は禁止。
- **Service**: 「業務の意味」を閉じ込める。
  - `QueryService`: 参照系（集計、フィルタリング）
  - `CommandService`: 操作系（作成、更新、削除）
  - `ConfirmService`: 状態遷移（確定処理）等
- **Repository**: 「どう取得・保存するか」のみを担当。Serviceからクエリビルダの詳細を隠蔽する。

### フロントエンド

- **Component**: Atomic Designをベースにしつつ、`shared/components/ui` に汎用パーツ、`features/` 内に機能特化したパーツを配置。
- **State Management**:
  - APIデータはすべて React Query で管理し、`staleTime` などを利用して最適化。
  - 複雑なフォームは React Hook Form + Zod で型安全にバリデーション。
- **Data Flow**: `UI -> Custom Hook -> API Client (fetch wrapper) -> Server`

## 開発・ビジネスルール

- **プロジェクト構造の自動認識**: コマンドを実行する前に、必ずカレントディレクトリと `backend/`, `frontend/` の構造を確認してください。
- **Docker/Sailの使用**:
  - `backend` 配下の `artisan` や `phpunit` などのコマンドは、原則として Docker コンテナ内で実行してください。
  - 基本的に `docker compose -f backend/compose.yaml exec backend [command]` または `sail` 経由で実行することを優先してください。
  - 直接ホストマシンで PHP コマンドを実行しないでください。
- **Dockerの自動起動**:
  - もし Docker コンテナが起動していない場合は、プロジェクトルートにある `scripts/docker-start.sh` を実行して各サービスを起動してください。
  - Docker Desktop 自体が起動していないことが推測される場合は、環境に応じたコマンド（例: Linux なら `sudo systemctl start docker` 等）を試みるか、ユーザーに確認してください。
- **依存関係の確認**: コマンドが失敗した場合は、エラーメッセージから「コンテナが起動していない」「ディレクトリが違う」などの原因を推測し、ユーザーに聞く前にまず自分で `ls` などのツールで状況を確認してください。

### 1. 認証フロー (Sanctum)

- **認証シーケンス**: ログイン前に `GET /sanctum/csrf-cookie` を呼び出し、次に `POST /api/login` を叩くフローを厳守。
- **認証状況判定**: ユーザー情報取得APIが成功するかどうかでログイン状態を判定。

### 2. シフトドメインの仕様

- **期間概念**: 半月単位（1〜15日、16日〜末日）での管理。
- **状態遷移**: `draft` (下書き) → `confirmed` (確定)。
- **編集ロック**: `confirmed` 状態のシフトは原則として編集不可（PolicyやRequestで制限）。
- **ガント表示**: スタッフ × 日付のグリッド表示。日付の範囲計算には `date-fns` を使用。

## ディレクトリ・ファイル命名規則

- **Backend (PHP)**: PSR-4 準拠。`ShiftService.php`, `ShiftRepository.php` (PascalCase)。
- **Frontend (React)**:
  - Component: `ShiftTable.tsx` (PascalCase) / ディレクトリは `shift-table/` (kebab-case)
  - Hook: `useShifts.ts` (camelCase + use prefix)
  - Utils: `formatDate.ts` (camelCase)

## テスト戦略

- **Phase1**: 「テスト可能な設計（CSR）」を優先。主要なServiceロジックのUnitテストと、認証/基本CRUDのFeatureテスト。
- **Phase2**: CI (GitHub Actions) 導入、Featureテストの拡充、フロントエンドの主要ロジックに Vitest を導入。

## アンチパターン

- **Controller内の肥大化**: ビジネスロジックを直接書かない。
- **Service内でのEloquent型依存**: `where`句などの詳細クエリをServiceに書かず、Repositoryに任せる。
- **useEffectの濫用**: データフェッチは React Query、イベントハンドラで済む処理に `useEffect` は使わない。
- **anyの禁止**: TypeScript では可能な限り型を定義する。

## 実装前の必読ドキュメント

- [Phase1 APIルート設計書.md](../docs/Step1_準備/Step1-3_構造設計/Phase1-API-ルート設計書.md)
- [Controller Service Repository 責務割り当て.md](../docs/Step1_準備/Step1-3_構造設計/Controller-Service-Repository-責務割り当て.md)
- [主要テーブル定義草案.md](../docs/Step1_準備/Step1-3_構造設計/主要テーブル定義草案.md)
- [難易度の高いタスク確認と実装方法.md](../docs/Step1_準備/Step1-1_機能要件定義/難易度の高いタスク確認と実装方法.md)
- [テスト駆動開発（TDD）とPhase別テスト戦略の整理.md](../docs/Step1_準備/Step1-4_非機能要件・品質設計/テスト駆動開発（TDD）とPhase別テスト戦略の整理.md)

## コパイロットへの追加指示

- コマンドを提案する際は、必ず以下の2点を明記してください。
  1. 目的：そのコマンドを実行する理由
  2. 具体的な処理内容：そのコマンドによって生じる具体的な変化
- コミットメッセージは必ず日本語で生成すること。
- 変更を行った場合は、提案するコミットメッセージをチャット欄にも必ず出力すること。
- コミットメッセージは「接頭辞: 本文」「理由: 〜」の形式で出力すること（例: refactor: 未使用依存の除去とCSSテーマ変数をデフォルト命名に統一 理由: 実装で参照されていない依存の削除と、shadcn/uiの標準に合わせたため）。

---
> Source: [takuya-ya/shifty](https://github.com/takuya-ya/shifty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-07 -->
