# GitHub Copilot Instructions for PhotoGeoExplorer

> [!IMPORTANT]
> このファイルは `AgentGuidelineSource.md` を正本として `scripts/Sync-AgentDocs.ps1` で自動生成されています。
> 共通ルールの変更は `AgentGuidelineSource.md` を編集し、`pwsh ./scripts/Sync-AgentDocs.ps1` を実行してください。
> 手編集は `<!-- BEGIN TOOL-SPECIFIC -->` と `<!-- END TOOL-SPECIFIC -->` の間だけ許可します。

## 対象
- GitHub Copilot Coding Agent

## 共通ガイドライン（自動生成）
<!-- BEGIN SHARED -->
# AIエージェント共通ガイドライン（正本）

このファイルは、`AGENTS.md` / `CLAUDE.md` / `.github/copilot-instructions.md` の共通ルールを管理する正本（Single Source of Truth）です。

## 基本方針
- AI エージェントの指示は分散管理しません。共通ルールはこのファイルのみを更新します。
- 共通ルール更新後は `pwsh ./scripts/Sync-AgentDocs.ps1` で配布先ファイルを再生成します。
- 配布先ファイルで手編集を許可するのは「固有補足」ブロックのみです。

## 言語とコミュニケーション
- このリポジトリでは日本語を使用します。
- AI エージェントの返答は日本語で行います。
- ドキュメントは日本語で記述します。
- コミットメッセージは日本語で記述します。
- 英語ドキュメントを見つけた場合は、日本語版を作成して置き換えます。

## プロジェクト概要
- PhotoGeoExplorer は、写真の位置情報（GPS/EXIF）を地図上に表示する Windows デスクトップアプリです。
- 地図描画は Mapsui を使用します。
- WebView2 は主にヘルプ表示などの補助用途で使用します。
- 配布は Microsoft Store（MSIX）を基準とし、MSI インストーラー配布は廃止済みです。

## プロジェクト構成
```text
PhotoGeoExplorer.sln
├─ PhotoGeoExplorer/        # WinUI Shell + Panes
│  ├─ MainWindow.xaml(.cs)  # Shell（薄く保つ）
│  ├─ Panes/                # 機能別UI
│  ├─ Services/             # アプリ/ドメインサービス
│  └─ State/                # ペイン間共有状態
├─ PhotoGeoExplorer.Core/   # コアロジック
├─ PhotoGeoExplorer.Tests/  # xUnit
└─ PhotoGeoExplorer.E2E/    # E2E
```

## MainWindow肥大化防止ガードレール
- MainWindow は Shell 責務（レイアウト、DI、ライフサイクル）に限定します。
- 新規機能ロジックは `Panes/*ViewModel` または `Services` に実装し、MainWindow へ直接追加しません。
- MainWindow にイベントハンドラを追加する場合は「委譲のみ」に留め、処理本体は別層へ移譲します。

## MVVM 責務ガードレール

View（`*.xaml.cs`）は UI 表示・入力受付に限定し、ビジネスロジックを書かないこと。

### View に書いてよいもの（純粋な UI 責務）
- ダイアログ・フライアウト・ピッカーの表示と結果の受け取り
- コンテキストメニューやアニメーションの構築
- ビジュアルツリー操作（`FindAncestor` 等）
- WinRT Interop が必要な処理（`FolderPicker` の HWND 初期化など）
- イベントハンドラ（処理本体は ViewModel/Service へ委譲する）

### View に書いてはいけないもの（ViewModel か Service で実装する）
- `foreach` ループでファイルを処理する操作ロジック（移動・削除・リネーム等）
- 成功 / 失敗 / スキップのカウント集計
- パス検証・比較（`IsSamePath`, `IsDescendantPath` 等）
- ファイル名の正規化・バリデーション
- `CancellationTokenSource` の作成・管理
- ステータスバーへの進捗・サマリー表示ロジック
- `CancellationToken` を受け取るループ処理全般

### 連携パターン（View と ViewModel の正しい分担）

複数ファイル操作においては以下のパターンを使うこと：

```csharp
// View（ダイアログ表示のみ担当）
private async Task MoveSelectionAsync()
{
    var targetPath = await PickFolderAsync();          // UI: フォルダ選択
    if (targetPath is null) return;

    await ViewModel.ExecuteMoveOperationAsync(         // ロジック: ViewModel に委譲
        targetPath,
        ViewModel.SelectedItems,
        ShowMoveConflictDialogAsync);                  // UI コールバック: 競合ダイアログ
}

// ViewModel（ループ・カウント・キャンセル管理を担当）
// ConflictResolution は ViewModel 内 internal enum として定義（View↔ViewModel 境界を跨いで参照するため）
internal async Task ExecuteMoveOperationAsync(
    string targetPath,
    IReadOnlyList<PhotoListItem> items,
    Func<string, bool, Task<ConflictResolution>> confirmCallback)
{
    // foreach・キャンセル判定・カウント集計はここで行う
}
```

> **判断基準**: そのコードを「ViewModel の単体テストで検証できるか？」を問いかける。
> できなければ View に置くべきではなく、ViewModel または Service へ移動する。

### ViewModel に書いてはいけないもの（Service に委譲する）
- ファイルシステムへの直接アクセス（`System.IO` 呼び出し）
- パス構造の解析・変換ロジック（パス結合・親フォルダ取得・拡張子操作等）
- ファイル名生成ルール（連番付与・重複回避・正規化等）
- 副作用を伴うビジネスロジック（ファイル移動・削除・リネームの実処理）

> **判断基準**: そのコードを「Service の単体テストで検証できるか？」を問いかける。
> ファイルシステムや外部リソースに直接触れる処理は Service へ移動する。

## ビルド・実行・テスト
- 通常ビルド: `dotnet build PhotoGeoExplorer.sln -c Release -p:Platform=x64`
- ローカル実行: `dotnet run --project PhotoGeoExplorer/PhotoGeoExplorer.csproj -c Release -p:Platform=x64`
- テスト実行: `dotnet test PhotoGeoExplorer.sln -c Release -p:Platform=x64`
- フォーマット: `dotnet format PhotoGeoExplorer.sln`

### パッケージ作成・インストール（MSIX）
- `.\\scripts\\DevInstall.ps1`: ビルド、署名、インストールを一括実行（既定でリビルド）
- `.\\scripts\\DevInstall.ps1 -ReuseBuild`: 既存ビルドの再インストール・署名のみ
- `.\\scripts\\DevInstall.ps1 -Clean`: アンインストールとクリーンアップ

### WACK テスト
- `.\\scripts\\RunWackTests.ps1`: ローカルインストールパッケージに対して WACK テストを実行


## コーディングスタイルと命名規則
- C# は 4 スペースインデント。既存の `.editorconfig` と既存実装に合わせます。
- 型と公開メンバーは `PascalCase`、ローカル/引数は `camelCase`、private フィールドは `_field` 形式です。
- public / internal / protected の識別子にアンダースコアは使いません（CA1707 対応）。
- 非同期メソッド名は `Async` で終えます（例: `InitializeMapAsync`）。
- XAML 要素名は `PascalCase` を使用します（例: `MapStatusText`）。
- 解析は厳格です（`AnalysisLevel=latest`、警告はエラー扱い）。

## テスト方針
- テストプロジェクトは `PhotoGeoExplorer.Tests`（xUnit）と `PhotoGeoExplorer.E2E` です。
- E2E を実行する場合は `PHOTO_GEO_EXPLORER_RUN_E2E=1` を指定します。

## コミット・PR ガイドライン
- コミットメッセージはコンベンショナルコミット形式を使います。
- 依存更新は `chore(deps): ... (#NN)`（Renovate 形式）を使います。
- PR には要約、理由、検証方法（コマンド/ログ）を含めます。
- UI 変更はスクリーンショットを添付し、関連 Issue があればリンクします。
- ソースコードまたはドキュメントなどプロジェクト成果物に変更を加えた場合は、`CHANGELOG.md` の更新を **PR に含めます**（マージ後の後追い更新は不可）。
- `tasks.md` など作業管理ファイルの更新も同様に PR に含めます。

## ブランチ運用
- `main` へのブランチ保護ルールにより直接コミットは **完全禁止** です。すべての変更はサブブランチで作業し PR でマージします。
- 大きいタスクは先に Issue を作成して整理します。

## セキュリティと設定
- ログは `%LocalAppData%\\PhotoGeoExplorer\\Logs\\app.log` に出力し、起動時にリセットします。
- Release ワークフローは `v1.2.0` のようなタグで `win-x64` 向け MSIX Bundle（`.msixupload`）を作成します。
- バージョン更新時のチェック項目は `docs/ReleaseChecklist.md` を参照します。

## PR 作成後の自動レビュー対応ルーティン

詳細手順: `docs/pr-review-routine.md` を参照してください。

## バージョン調査の注意
- AI から見て不自然に新しいバージョンに感じても、勝手にバージョンダウンしないでください。
- 学習時期のタイムラグを前提に、必要に応じて Web で最新情報を確認します。
<!-- END SHARED -->

## 固有補足（手編集可）
<!-- BEGIN TOOL-SPECIFIC -->
- GitHub Copilot 固有の追加指示がある場合、このブロックに追記してください。
<!-- END TOOL-SPECIFIC -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/scottlz0310)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/scottlz0310)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
