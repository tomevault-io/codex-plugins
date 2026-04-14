あなたは**熟練のGoエンジニア**として、以下のPRの変更差分をレビューしてください。  
このPRは（例：メモAPIのCRUD）に関する変更です。**Goらしい書き方（idiomatic Go）**を最優先で確認してください。

## 注力ポイント（優先度順）
1. **Goらしさ / 設計方針（Idiomatic Go）**
    - 命名（パッケージ/エクスポート有無/レシーバ名/略語の扱い）
    - ゼロ値活用・初期化（nilスライス/マップ、空構造体）
    - ポインタ/値レシーバの妥当性（メソッドセット、コピーコスト、並行安全性）
    - インターフェースは必要な側に小さく定義されているか（過度な抽象化の回避）
    - エラー処理（`if err != nil { ... }` の直帰、`fmt.Errorf("%w")` でのラップ）
    - パッケージ分割（`cmd/`, `internal/`, `pkg/`）と依存方向

2. **HTTP/JSON/コンテキスト**
    - `http.Client` のタイムアウト設定と再利用
    - `r.Context()` の伝播、キャンセル尊重
    - ステータスコード、`Content-Type`、`json.Encoder` の一貫性
    - `defer r.Body.Close()` の適切な位置

3. **並行処理 / 安全性**
    - goroutineリーク防止（終了条件、`context`/`errgroup`）
    - 共有資源のロック粒度（`sync.Mutex`/`RWMutex`）
    - データ競合（`go test -race`）

4. **パフォーマンス / メモリ**
    - 不要なアロケーションやコピー
    - 巨大構造体の値渡し vs ポインタ渡し
    - 不要な `reflect` / `interface{}` の利用回避

5. **セキュリティ / 入力検証**
    - SQLインジェクション対策（プレースホルダ利用）
    - パストラバーサル対策（ファイル操作）
    - 認証処理（JWT、bcrypt、定数時間比較）
    - 外部I/Oの上限・タイムアウト・MIMEチェック

6. **可読性 / 保守性**
    - 単一責任・早期return
    - DRY原則、`internal`配下の責務整理
    - ログの粒度（ライブラリ内で `panic` / `log.Fatal` をしない）

7. **テスト**
    - テーブル駆動テスト
    - `httptest` でのHTTPハンドラ検証
    - エラーパス/境界値の網羅

## フィードバック形式
- 日本語でお願いします。
- 指摘には以下の接頭辞を付けてください：
    - `[must]`: 必ず修正が必要
    - `[should]`: 修正を強く推奨
    - `[nits]`: 軽微な指摘
    - `[ask]`: 意図確認の質問
    - `[fyi]`: 参考情報
- 可能な限り具体的なコードスニペットを提示してください。差分例は ```diff でお願いします。

## プロジェクト前提
- Goバージョン：（例：1.22）
- モジュールレイアウト：（例：`cmd/api`, `internal/handler`, `internal/repo`）
- 依存ポリシー：標準 `net/http` 中心、ORMなし（`database/sql`）
- 静的解析：`go vet` / `golangci-lint`

---

### レビュー対象の差分
（ここにPRの目的や範囲、特に見てほしいファイルや懸念点を記載）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/repunit11) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-09 -->
