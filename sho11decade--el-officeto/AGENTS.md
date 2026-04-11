<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# Office Image Extractor - Copilot Instructions

このプロジェクトは、Microsoft Officeファイル（Word、Excel、PowerPoint）から画像を抽出・表示・保存するElectronアプリケーションです。

## プロジェクト概要

- **技術スタック**: Electron, Node.js, HTML5, CSS3, JavaScript
- **主要ライブラリ**: mammoth.js, xlsx, yauzl, sharp, electron-builder
- **対象ファイル**: .docx, .doc, .xlsx, .xls, .pptx, .ppt

## アーキテクチャ

- **メインプロセス** (`src/main.js`): Electronのメインプロセス、ネイティブAPI処理
- **プリロード** (`src/preload.js`): セキュアなIPC通信のためのブリッジ
- **レンダラー** (`src/renderer/`): UI表示とユーザー操作処理
- **ユーティリティ** (`src/utils/`): 画像抽出とファイル処理

## コーディング規約

### JavaScript
- ES6+の機能を積極的に使用
- async/awaitを使用した非同期処理
- エラーハンドリングを適切に実装
- セキュリティを考慮したIPC通信

### CSS
- BEMに近い命名規則
- CSS GridとFlexboxを活用
- レスポンシブデザイン対応
- アクセシビリティを考慮

### セキュリティ
- `nodeIntegration: false`
- `contextIsolation: true`
- プリロードスクリプトによる安全なAPI公開

## 機能拡張時の考慮事項

1. **新しいOffice形式対応**: `src/utils/imageExtractor.js`に処理を追加
2. **UI改善**: `src/renderer/`配下のファイルを更新
3. **パフォーマンス**: 大容量ファイル処理時のメモリ使用量に注意
4. **エラーハンドリング**: ユーザーフレンドリーなエラーメッセージ

## テスト対象ファイル

開発時は以下のファイル形式でテストを実施：
- Word: .docx（推奨）、.doc（制限あり）
- Excel: .xlsx（推奨）、.xls（制限あり）  
- PowerPoint: .pptx（推奨）、.ppt（制限あり）

## 既知の制限

- .doc、.xls、.ppt形式は画像抽出が制限的
- SVG形式の画像は表示に制限がある場合あり
- 暗号化されたOfficeファイルは非対応

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sho11decade)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/sho11decade)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
