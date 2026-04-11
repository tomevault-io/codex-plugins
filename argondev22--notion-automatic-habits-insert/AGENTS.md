# GitHub Copilot PRレビュー指示

## 🎯 レビューの目的

このプロジェクトは**Notion APIを使用した習慣管理システム**です。高品質で保守性の高いコードを維持するため、以下の観点でレビューしてください。

## 🌏 言語設定

**重要**: すべてのレビューコメントは**日本語**で記述してください。コードの説明、改善提案、質問など、すべて日本語で行ってください。

## 🏗️ アーキテクチャ原則

### 1. **レイヤードアーキテクチャの遵守**

- **Presentation Layer**: `fetch.ts` - 外部API
- **Business Layer**: `HabitRepository` - ビジネスロジック
- **Service Layer**: `NotionDatabaseService` - サービス層
- **Data Access Layer**: `HabitMapper` - データ変換
- **Infrastructure Layer**: Core services (Cache, Logger, etc.)

### 2. **依存性注入パターン**

```typescript
// ✅ 良い例: DIコンテナを使用
const repository = ServiceFactory.getService<HabitRepository>("habitRepository");

// ❌ 悪い例: 直接インスタンス化
const repository = new HabitRepository();
```

### 3. **ファクトリーパターン**

- すべてのサービスは`ServiceFactory`経由で取得
- 直接的な`new`によるインスタンス化を避ける

## 🔍 コード品質チェック項目

### **TypeScript関連**

- [ ] **型安全性**: `any`型の使用を避け、適切な型定義
- [ ] **strict mode**: TypeScriptのstrict設定を遵守
- [ ] **インターフェース**: 明確な契約定義
- [ ] **ジェネリクス**: 型安全な再利用可能なコード

### **エラーハンドリング**

- [ ] **カスタムエラー**: `FetchError`クラスの適切な使用
- [ ] **エラーコード**: 構造化されたエラー管理
- [ ] **型安全**: コンパイル時エラー検出
- [ ] **ログ出力**: 適切なエラーログ記録

### **パフォーマンス**

- [ ] **キャッシュ戦略**: メモリ効率的なキャッシュ使用
- [ ] **並列処理**: `Promise.all`/`Promise.allSettled`の適切な使用
- [ ] **リトライ戦略**: 指数バックオフの実装
- [ ] **メモリリーク**: 適切なリソース解放

### **セキュリティ**

- [ ] **入力バリデーション**: すべての外部入力の検証
- [ ] **機密情報**: APIキーやトークンの適切な管理
- [ ] **エラー情報**: 機密情報の除外

## 📋 具体的なレビューポイント

### **1. 新機能追加時**

```typescript
// ✅ 良い例: レイヤー分離
export class NewFeatureService {
  constructor(
    private logger: Logger,
    private cache: Cache,
    private validator: Validator
  ) {}
}

// ❌ 悪い例: レイヤー混在
export class NewFeatureService {
  async fetchData() {
    // 直接Notion APIを呼び出し
    const response = await notion.pages.retrieve({ page_id: id });
  }
}
```

### **2. エラーハンドリング**

```typescript
// ✅ 良い例: 構造化されたエラー
try {
  const result = await someOperation();
  return { success: true, data: result };
} catch (error) {
  throw new FetchError("操作に失敗しました", ERROR_CODES.OPERATION_FAILED, {
    originalError: error,
  });
}

// ❌ 悪い例: 生のエラー
try {
  const result = await someOperation();
  return result;
} catch (error) {
  throw error; // 情報が不足
}
```

### **3. 設定管理**

```typescript
// ✅ 良い例: 設定の一元管理
const config = Config.getInstance();
const result = await service.operation(config.get("timeout"));

// ❌ 悪い例: ハードコード
const result = await service.operation(5000);
```

## 🚨 重要な注意点

### **絶対に避けるべきパターン**

1. **直接的なNotion API呼び出し** - 必ずService層を経由
2. **グローバル変数の使用** - DIコンテナを使用
3. **ハードコードされた値** - 設定ファイルから取得
4. **try-catchの不適切な使用** - カスタムエラークラスを使用
5. **console.logの直接使用** - Loggerクラスを使用

### **推奨パターン**

1. **単一責任の原則** - 各クラスは一つの責任のみ
2. **依存性逆転** - 抽象に依存、具象に依存しない
3. **開閉の原則** - 拡張に開いて、修正に閉じている
4. **インターフェース分離** - クライアントは不要なインターフェースに依存しない

## 🧪 テスト観点

### **単体テスト**

- [ ] **モック注入**: `ServiceFactory.reset()`を使用
- [ ] **エラーケース**: 異常系のテスト
- [ ] **境界値**: エッジケースのテスト

### **統合テスト**

- [ ] **実際のAPI**: Notion APIとの連携テスト
- [ ] **エンドツーエンド**: 全体フローのテスト

## 📊 パフォーマンス指標

### **チェック項目**

- [ ] **レスポンス時間**: 適切なタイムアウト設定
- [ ] **メモリ使用量**: キャッシュサイズの最適化
- [ ] **並列処理**: 効率的な非同期処理
- [ ] **リトライ回数**: 適切なリトライ戦略

## 🔧 ツール設定

### **ESLint/Prettier**

- [ ] **コードフォーマット**: Prettierの設定遵守
- [ ] **リンター**: ESLintルールの遵守
- [ ] **型チェック**: TypeScriptの型チェック通過

### **依存関係**

- [ ] **バージョン管理**: 適切なバージョン指定
- [ ] **セキュリティ**: 脆弱性のない依存関係
- [ ] **サイズ**: 不要な依存関係の排除

## 💡 レビュー時の質問

1. **この変更は既存のアーキテクチャに適合していますか？**
2. **エラーハンドリングは適切に実装されていますか？**
3. **パフォーマンスに影響はありませんか？**
4. **テストは十分にカバーされていますか？**
5. **セキュリティ上の問題はありませんか？**

## 📝 レビューコメントの書き方

### **日本語での記述例**

```text
✅ 良い例:
「この実装はレイヤードアーキテクチャに従っており、適切にService層を経由しています。」

❌ 改善提案:
「エラーハンドリングでFetchErrorクラスを使用することを推奨します。現在の実装では詳細なエラー情報が失われる可能性があります。」

🔍 質問:
「この関数のパフォーマンステストは実施されていますか?大量のデータ処理時の動作を確認する必要があります。」
```

---

**重要**: このプロジェクトは**高品質で保守性の高いコード**を重視しています。機能の実装だけでなく、**アーキテクチャの一貫性**と**コードの品質**を最優先でレビューしてください。**すべてのコメントは日本語で記述**してください。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/argondev22)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/argondev22)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
