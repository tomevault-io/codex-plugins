# Copilot Instructions

## 🧭 基本方針

- Python は使わないでって
- このプロジェクトは **Azure 上のデモ環境（掲示板アプリ + 管理アプリ + MySQL + バックアップ）をフル IaC + CI/CD で構築する**ことを目的とする。
- 説明・コメント・ドキュメントは **すべて日本語**で記述する。
- 必ず **Bicep + パラメータファイル（parameters/\*.json）** を使用する。
- 手動構築は禁止。フルコード化と疎結合化を徹底する。
- コストを抑えるため、Azure リソースは **低コスト SKU を最優先で選択**する。
- トラブルシューティング履歴はすべて Markdown で管理する。 パス : `trouble_docs/` 配下に作成する

---

## 📦 必須 Azure リソース（低コスト設計）

Copilot は以下を優先としてコード生成する：

## ✅ 必須 Azure リソース（低コスト設計 + ACR 追加）

### ✔ 1. Azure Kubernetes Service（AKS：掲示板アプリ）

- 低コスト構成
  - NodePool：**B2s / B1ms / Standard_DS2_v2**
  - ノード数：**1 ノード**（デモ用途）
  - Standard Load Balancer（Basic は非推奨）
- アプリ公開は  
  **LB → NGINX Ingress → Pod（HTTP/HTTPS）**

---

### ✔ 2. Azure Container Apps（ACA：DB 管理アプリ）

- Dedicated ではなく **Consumption（Serverless）** を最優先
- Basic 認証（ID ＋パスワード）で簡易保護

---

### ✔ 3. Azure Container Registry（ACR：コンテナイメージ管理）

- 掲示板アプリ・管理アプリのコンテナイメージをビルドして ACR にプッシュ
- SKU は **Basic**（最低コスト）
- GitHub Actions から **Service Principal 認証**でログイン
- ACR → AKS / ACA への Pull 権限を付与（`AcrPull`）

---

### ✔ 4. Azure VM（MySQL DB）

- 最低限の VM サイズ（例：**B1ms / B2s**）
- Managed Disk は **Standard HDD**
- MySQL・バックアップスクリプト・Log Analytics Agent をセットアップ

---

### ✔ 5. Storage Account（バックアップ）

- 冷蔵（Cool）または Archive Tier を検討
- ストレージアカウントは **Standard LRS（最低コスト）**

---

### ✔ 6. Log Analytics Workspace

- すべてのログ & メトリクスを **LA Workspace に送信**
- 収集対象：
  - AKS（ContainerInsights + Control plane logs）
  - ACA（Logs / Console output）
  - VM（Syslog / MySQL log / cron log）
  - Storage Diagnostics
  - AzureActivity（必須）

---

## 📁 ディレクトリ設計（統一ルール）

infra/
main.bicep
modules/
parameters/
main-dev.parameters.json
app/
board-app/
src/
k8s/
dummy-secret.txt # アプリからブラウザで確認できるようにルーティング
admin-app/
src/
.github/workflows/

---

## 🔐 dummy-secret.txt のルーティング要件

Copilot は必ず以下を実装する：

- `dummy-secret.txt` は **アプリの `public/` もしくは `/static/` 相当の場所に配置**
- 掲示板アプリのトップページに  
  **「ダミーシークレットはこちら」** のリンクを貼る
- HTTP で直接アクセス可能  
  例： GET /dummy-secret

- ダミーであり本物の鍵ではない旨をコメントに明記する

---

## 🔐 認証（Service Principal + Client Secret）

- GitHub Actions → Azure の認証は **Service Principal (Client Secret)** を使用
- Secrets/Variables:
  - `AZURE_CLIENT_ID` (Variables)
  - `AZURE_TENANT_ID` (Variables)
  - `AZURE_CLIENT_SECRET` (Variables)
  - `AZURE_SUBSCRIPTION_ID` (Secrets)
- コード内に Subscription ID・GUID を埋め込まない

---

## ⚙️ CI/CD ワークフロー（疎結合）

Copilot は以下の 7 つのワークフロー生成を前提とする：

1. **infra-deploy.yml**

   - Bicep Validate → What-If → Deploy
   - **parameters.json** を読み込んでデプロイする
   - AKS・ACA・VM・Storage・LAW を作成

2. **app-build-board.yml**

   - 掲示板アプリをコンテナビルド
   - Trivy Image Scan
   - ACR push（権限は workload identity）

3. **app-deploy-board.yml**

   - AKS へデプロイ（kubectl apply）
   - Ingress / Service / Deployment
   - dummy-secret.txt の公開ルート追加

4. **app-build-admin.yml**

   - ACA アプリビルド
   - Trivy Image Scan
   - ACR push

5. **app-deploy-admin.yml**

   - ACA へリビジョンデプロイ
   - 認証設定（ID + PW）

6. **backup-upload.yml**

   - 1 時間おき
   - VM → Storage へ MySQL バックアップをアップロード
   - cron の成功ログを LA へ送信

7. **cleanup-failed-workflows.yml**
   - 1 時間おき
   - GitHub Actions の failed 状態や lock をクリーンアップ
   - 一時ファイルやキャッシュも削除

---

## 🛡️ セキュリティスキャン

- CodeQL（SAST）
- Trivy（コンテナ & IaC）
- Secret leakage（Gitleaks / Secret scanning）
- Dependabot（SCA）

---

## 📊 ログ収集と送信（必須仕様）

Copilot は以下を常に含める：

### 1. AKS

- ContainerInsights（Azure Monitor agent）
- Control Plane logs
- kube-apiserver
- kube-scheduler
- kube-controller-manager
- Storage Account の診断ログ
- azureactivity
- Azure VM の Log Analytics Agent

### 2. ACA

- Console logs
- System logs
- Request/response logs

### 3. VM（MySQL）

- Syslog
- MySQL error log
- cron 実行ログ
- VM エージェントの Heartbeat

### 4. Storage Account

- StorageAccountLogs → LA Workspace

---

## 🧱 設計ルール

- Bicep はすべての定数を **parameters.json** へ逃がす
- VNet / サブネット / IP 範囲 もハードコード禁止
- コメントは「なぜこの設計が必要か」を必ず書く
- 冪等性を最優先
- リソースは **最低コスト SKU** を基本にする

---

## 📝 Copilot の最優先指針

Copilot は生成時に必ず以下を守る：

1. **フルコード化（IaC + CI/CD）**
2. **低コスト設計**
3. **マジックナンバー禁止**
4. **セキュア（Service Principal / 最小権限 / スキャン必須）**
5. **ログ収集を LA Workspace に統合**
6. **dummy-secret を UI で確認できる状態で配置**
7. **疎結合ワークフロー**
8. **再現性（他者環境で動く）**
9. **コマンドはユーザー確認なしで即時実行可（CI/CD 実行・検証の迅速性を優先）**
10. **Azure CLI / Azure PowerShell / GitHub CLI の利用を標準手段として明記**

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/erikaambiru)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/erikaambiru)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
