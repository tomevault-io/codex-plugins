# Copilot Instructions — 個人タスク管理リポジトリ

## リポジトリの目的

このリポジトリは GitHub Issues と GitHub Projects を使った **個人タスク管理** の場です。
コードは含まず、Issue の管理・追跡のみを行います。

## 用語と構造

### Goal（目標）

- 達成したい目標を表す Issue。
- ラベル: `goal`
- 例: 「ブログのリニューアル」「AWS 認定資格の取得」

### Task（タスク）

- Goal を達成するための具体的な作業単位を表す Issue。
- ラベル: `task`
- 例: 「デザインカンプの作成」「模擬試験を 3 回受ける」

### Goal と Task の関係

- Goal と Task は **sub-issue** で紐づける（Task が Goal の sub-issue）。
- 1 つの Task は 1 つの Goal にのみ属する。
- Goal に属さない独立した Task も許容する。

## ラベル規約

| ラベル | 用途 |
|--------|------|
| `goal` | Goal Issue に付与 |
| `task` | Task Issue に付与 |

これ以外のラベルは自由に追加してよい（カテゴリ分類など）。

## ステータス管理（GitHub Projects）

ステータスは GitHub Projects の **Status フィールド** で管理する。Issue の open/closed とは別に、以下のステータスを使う。

### Task のステータス

| ステータス | 意味 |
|------------|------|
| *(無印)* | まだ未着手・未分類 |
| **ToDo** | やると決めたもの |
| **In Progress** | 現在作業中 |
| **Done** | 完了 |

- Task が Done になったら Issue も close する。
- すべての Task が Done になった Goal は close を検討する。

### Goal のステータス

Goal のステータスも同じ Projects の Status フィールドで管理する。Task と同じステータス値を使用する。

## 締め切り管理（Due Date）

GitHub Projects の **Due Date** カスタムフィールド（Date 型）で締め切りを管理する。Goal・Task どちらにも設定可能。

```bash
# Due Date の設定
gh project item-edit --project-id "<プロジェクト ID>" --id "<アイテム ID>" --field-id "<Due Date フィールド ID>" --date "YYYY-MM-DD"
```

- プロジェクト ID・フィールド ID は `gh project field-list <番号> --owner runceel --format json` で取得可能。
- アイテム ID は `gh project item-list <番号> --owner runceel --format json` で取得可能。

## Issue の書き方

### Goal Issue

```
タイトル: 目標を簡潔に表す文
本文:
- 背景・動機（なぜこの Goal を立てたか）
- 完了条件（何をもって達成とするか）
```

### Task Issue

```
タイトル: 作業内容を簡潔に表す文
本文:
- やること（具体的な作業内容）
- 完了条件（何をもって Done とするか）
- 備考（参考リンク、注意点など。任意）
```

## コメントの書き方

Goal や Task の進捗・作業メモは Issue のコメントとして記録する。

### フォーマット

コメントの 1 行目に種別を示すプレフィックスを付け、続けて本文を記載する。本文は必須。

```
[種別] 本文
```

### 種別一覧

| プレフィックス | 用途 |
|----------------|------|
| `[進捗]` | 作業の進捗や気づきの記録 |
| `[完了報告]` | Done にした際のまとめ |
| `[メモ]` | その他の備忘録 |

### ルール

- Task を Done にする際は `[完了報告]` コメントを必ず残す（何をやったかの簡単なまとめ）。
- プレフィックスは上記以外を自由に追加してもよい。

### コマンド例

```bash
gh issue comment <番号> --body "[進捗] デザインの初稿が完了。レビュー待ち。"
gh issue comment <番号> --body "[完了報告] 全ページのデザインカンプを作成し Figma に共有済み。"
```

## gh CLI の利用

操作には `gh` CLI を使う。

### ヘルパースクリプト

`scripts/` ディレクトリに、GitHub Projects からデータを取得するヘルパースクリプトがある。
タスクやゴールの一覧取得には **これらのスクリプトを優先的に使う**。

スクリプトはリポジトリにリンクされた GitHub Project を**自動検出**する（ハードコード不要）。
複数プロジェクトがリンクされている場合は `-ProjectNumber` で指定する。

```powershell
# タスク一覧（全件）
.\scripts\Get-Tasks.ps1

# ステータスで絞り込み
.\scripts\Get-Tasks.ps1 -Status "In Progress"
.\scripts\Get-Tasks.ps1 -Status "Todo","In Progress"

# ゴール一覧
.\scripts\Get-Goals.ps1
.\scripts\Get-Goals.ps1 -Status "In Progress"

# プロジェクト番号を明示指定（複数リンク時）
.\scripts\Get-Tasks.ps1 -ProjectNumber 7
```

出力カラム: Number, Status, Title, DueDate, State

スクリプト内部で UTF-8 エンコーディング設定と GraphQL クエリを処理しているため、文字化けや複雑なクエリ組み立ては不要。

**プロジェクトが未リンクの場合**: スクリプトは `NO_LINKED_PROJECT` エラーを出力する。
この場合は `project-setup` エージェントを使ってプロジェクトを作成する。

### 文字化けへの対策（Windows 環境）

ヘルパースクリプトを使わず直接 `gh` CLI を呼ぶ場合、PowerShell で日本語が文字化けすることがある。
以下のいずれかで対処する:

```powershell
# 方法 1: OutputEncoding を設定してから ConvertFrom-Json（スクリプトではこの方法を採用）
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8
$result = gh api graphql -f query=$query 2>&1
$parsed = ($result | Out-String) | ConvertFrom-Json

# 方法 2: Python でパースする
gh api graphql -f query='...' | python -c "import sys,json; data=json.load(sys.stdin); ..."
```

### データ取得のルール

- `gh issue list` には常に `--limit 500` を付与し、デフォルトの 30 件制限を回避する。
- `--state` はデフォルト（open のみ）を基本とし、closed の情報が必要な場合のみ `--state closed` や `--state all` を使う。
  - open のみで十分な場面: 一覧表示、次にやるタスクの提案
  - closed が必要な場面: 進捗率の計算、日次・週次レビューでの完了タスク集計

### 主なコマンド例

```bash
# Issue 作成
gh issue create --title "タイトル" --body "本文" --label "goal"
gh issue create --title "タイトル" --body "本文" --label "task"

# Issue 一覧（open のみ）
gh issue list --label "goal" --limit 500
gh issue list --label "task" --limit 500

# Issue を close
gh issue close <番号>
```

### Sub-issue の操作

`gh issue update --add-sub-issue` は利用できない。代わりに **GraphQL API** を使う。

```bash
# Task を Goal の sub-issue として追加
# まず Issue の node ID を取得
gh issue view <番号> --json id --jq '.id'

# GraphQL mutation で紐づけ
gh api graphql -f query='
mutation {
  addSubIssue(input: {
    issueId: "<Goal の node ID>"
    subIssueId: "<Task の node ID>"
  }) {
    issue { id }
  }
}'
```

### 親 Goal の取得

Task から親 Goal を取得するには **REST API** の `parent_issue_url` フィールドを使う。
GraphQL には `parentIssue` フィールドが存在しないため、REST API 経由で取得する必要がある。

```bash
# Task の親 Goal の API URL を取得
gh api repos/{owner}/{repo}/issues/<Task番号> --jq '.parent_issue_url'

# 親 Goal の詳細を取得
gh api repos/{owner}/{repo}/issues/<Task番号> --jq '.parent_issue_url' | xargs gh api --jq '{number, title, state}'
```

### Goal の子 Task 一覧の取得

```bash
# Goal の sub-issues（子 Task）を一覧取得
gh api repos/{owner}/{repo}/issues/<Goal番号>/sub_issues --jq '.[] | {number, title, state}'
```

### Projects のステータス取得

GitHub MCP Server のツールでは Projects V2 のカスタムフィールド（Status 等）を取得できない。

**ステータス・Due Date 付きの一覧取得にはヘルパースクリプトを使う（推奨）:**

```powershell
.\scripts\Get-Tasks.ps1                        # 全タスク（Status, DueDate 付き）
.\scripts\Get-Tasks.ps1 -Status "In Progress"  # ステータスで絞り込み
.\scripts\Get-Goals.ps1 -Status "In Progress"  # ゴールも同様
```

### ステータス更新

Issue のステータス変更にはヘルパースクリプトを使う:

```powershell
# ステータスを変更（Todo / In Progress / Done）
.\scripts\Set-ItemStatus.ps1 -IssueNumber 5 -Status "Done"
.\scripts\Set-ItemStatus.ps1 -IssueNumber 8 -Status "In Progress"
.\scripts\Set-ItemStatus.ps1 -IssueNumber 10 -Status "Todo"
```

- Done に変更すると Issue を自動 close する。
- Done 以外に変更すると、Issue が closed の場合は自動 reopen する。
- プロジェクトの自動検出に対応（`-ProjectNumber` で明示指定も可）。

スクリプトでカバーできないケース（フィールド ID 取得など）では `gh api graphql` を直接使う。
プロジェクト番号は **7**（個人タスク管理デモ）。

---
> Source: [runceel/copilot-task-management](https://github.com/runceel/copilot-task-management) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-24 -->
