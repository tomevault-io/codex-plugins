# Copilot Instructions for Security Hub Bedrock Alert System

## プロジェクト概要

このプロジェクトは、AWS Security Hub のセキュリティ検出を Amazon Bedrock で日本語要約し、SNS 経由でアラート通知を行うサーバーレスシステムです。

### 技術スタック
- **インフラ**: AWS CDK v2 (TypeScript)
- **Lambda**: Python 3.11
- **AI**: Amazon Bedrock (Claude 4 Sonnet Inference Profile)
- **通知**: Amazon SNS
- **イベント処理**: Amazon EventBridge

## コーディング規約とベストプラクティス

### TypeScript/CDK コード

#### 命名規約
- **Stack名**: `SecurityhubBedrockAlertStack` (PascalCase)
- **Construct ID**: `SecurityHubAlertTopic`, `SecurityHubProcessor` (PascalCase)
- **変数名**: `alertTopic`, `lambdaRole`, `securityHubProcessor` (camelCase)
- **定数**: `MODEL_ID` (SCREAMING_SNAKE_CASE)

#### CDK ベストプラクティス
```typescript
// ✅ 推奨: 明示的なタイプと設定
const alertTopic = new sns.Topic(this, 'SecurityHubAlertTopic', {
  topicName: 'security-hub-bedrock-alerts',
  displayName: 'Security Hub Bedrock Alerts'
});

// ✅ 推奨: 最小権限の原則
lambdaRole.addToPolicy(new iam.PolicyStatement({
  effect: iam.Effect.ALLOW,
  actions: ['bedrock:InvokeModel'],
  resources: [/* 特定のリソースARN */]
}));

// ❌ 避ける: ワイルドカード権限
// resources: ['*']
```

#### インポート順序
```typescript
// 1. AWS CDK core
import * as cdk from 'aws-cdk-lib';
import { Construct } from 'constructs';

// 2. AWS CDK services (アルファベット順)
import * as events from 'aws-cdk-lib/aws-events';
import * as iam from 'aws-cdk-lib/aws-iam';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as sns from 'aws-cdk-lib/aws-sns';

// 3. Node.js built-in
import * as path from 'path';
```

### Python/Lambda コード

#### 命名規約
- **関数名**: `lambda_handler`, `generate_summary_with_bedrock` (snake_case)
- **変数名**: `finding_id`, `control_id`, `resources_text` (snake_case)
- **定数**: `MODEL_ID`, `SNS_TOPIC_ARN` (SCREAMING_SNAKE_CASE)

#### エラーハンドリング
```python
# ✅ 推奨: 明確なエラーハンドリング
try:
    summary = generate_summary_with_bedrock(finding)
    logger.info(f"Summary generated for finding: {finding_id}")
except Exception as e:
    logger.error(f"Error generating summary: {str(e)}")
    raise Exception(f"Bedrock summary generation failed: {str(e)}")

# ❌ 避ける: 曖昧なエラーハンドリング
# except:
#     pass
```

#### ログ出力規約
```python
# ✅ 推奨: 構造化ログ
logger.info(f"Processing finding: {finding_id} - {finding_title} (Control: {control_id})")
logger.error(f"Error processing finding {finding_id}: {str(e)}")

# ✅ 推奨: レベル別使い分け
logger.debug("Debug information")  # 開発時のみ
logger.info("Normal operation")    # 正常な処理フロー
logger.warning("Recoverable issue") # 軽微な問題
logger.error("Error occurred")     # エラー発生
```

#### 型ヒント
```python
# ✅ 推奨: 明示的な型ヒント
def lambda_handler(event: Dict[str, Any], context: Any) -> Dict[str, Any]:
def generate_summary_with_bedrock(finding: Dict[str, Any]) -> str:
def send_to_sns(summary: str, finding: Dict[str, Any]) -> None:
```

## プロジェクト固有の知識

### Security Hub データ構造
```python
# Finding の重要なフィールド
finding = {
    "Id": "arn:aws:securityhub:...",           # 一意識別子
    "Title": "Security control description",   # タイトル
    "Description": "Detailed description",     # 詳細説明
    "Severity": {"Label": "HIGH"},            # 重要度
    "ProductFields": {"ControlId": "EC2.19"}, # コントロールID
    "Compliance": {"SecurityControlId": "EC2.19"}, # セキュリティコントロールID
    "Resources": [...],                       # 影響リソース
    "AwsAccountId": "123456789012",          # アカウントID
    "Region": "ap-northeast-1"               # リージョン
}
```

### Bedrock プロンプト設計
```python
# ✅ 推奨: 構造化されたプロンプト
prompt = f"""
AWS Security Hubで以下のセキュリティ検出が報告されました。日本語で専門的な分析と要約を提供してください。

【検出情報】
- タイトル: {title}
- 説明: {description}  
- 重要度: {severity}
- セキュリティコントロールID: {control_id}
- 影響リソース: {resources_text}

【必須回答形式】
- 検出内容の要約: (...)
- 詳細説明: (...)
- 影響: (...)
- 対応の優先度: [Immediate/High/Medium/Low]
- 推奨される対応: (...)
- 参照リンク: (...)
"""
```

### EventBridge フィルターパターン
```typescript
// 重要度とステータスでフィルタリング
eventPattern: {
  source: ['aws.securityhub'],
  'detail-type': ['Security Hub Findings - Imported'],
  detail: {
    findings: {
      Compliance: {
        Status: ['FAILED', 'WARNING']  // 失敗・警告のみ
      },
      RecordState: ['ACTIVE'],         // アクティブのみ
      Severity: {
        Label: ['CRITICAL', 'HIGH', 'MEDIUM'] // 高・中重要度のみ
      },
      Workflow: {
        Status: ['NEW']                // 新規のみ
      }
    }
  }
}
```

## テスト方針

### 単体テスト
- Lambda 関数の各メソッドを個別にテスト
- モックを使用した外部サービス呼び出しのテスト
- エラーケースのテスト

### 統合テスト
- EventBridge → Lambda の連携テスト
- Lambda → Bedrock → SNS のエンドツーエンドテスト

## デプロイとCI/CD

### ローカル開発
```bash
# ビルド
npm run build

# CDK差分確認
npx cdk diff

# デプロイ
npx cdk deploy --region ap-northeast-1
```

### 環境固有設定
- **Development**: 東京リージョン（ap-northeast-1）
- **Production**: 複数リージョン対応
- **Bedrock Model**: `apac.anthropic.claude-sonnet-4-20250514-v1:0`

## セキュリティ考慮事項

### IAM 権限
- 最小権限の原則に従う
- 明示的なリソース ARN を指定
- クロスサービス権限は必要最小限に制限

### データ保護
- 機密情報のログ出力を避ける
- Security Hub データの適切な取り扱い
- SNS メッセージの暗号化を検討

## コメント規約

### TypeScript
```typescript
// SNSトピック作成
// Lambda IAMロール作成
// Bedrock権限を追加
```

### Python
```python
"""BedrockでSecurity Hubのfindingを日本語要約"""
# 基本情報を抽出
# コントロールIDを取得
# Bedrockプロンプト作成
```

## よくある問題と解決策

### Bedrock 権限エラー
```
原因: リージョン不一致や権限不足
解決: IAM ポリシーでの明示的な ARN 指定
```

### EventBridge フィルタリング
```
原因: 想定外のイベント形式
解決: detail 構造の詳細確認とフィルター調整
```

### SNS メッセージサイズ制限
```
原因: Bedrock 要約が長すぎる
解決: max_tokens の調整や要約の分割
```

## ファイル構造理解

```
securityhub-bedrock-alert/
├── lib/
│   └── securityhub-bedrock-alert-stack.ts  # CDK スタック定義
├── lambda/
│   ├── handler.py                          # Lambda メイン処理
│   └── requirements.txt                    # Python 依存関係
├── bin/
│   └── securityhub-bedrock-alert.ts        # CDK エントリーポイント
├── generated-diagrams/                     # システム構成図
├── SPECIFICATION.md                        # 技術仕様書
└── package.json                           # Node.js 設定
```

---

**重要**: このプロジェクトは Security Hub の機密データを扱うため、セキュリティ要件を常に最優先に考慮してコードを作成してください。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/leomaro7)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/leomaro7)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
