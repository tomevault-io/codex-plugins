# MarkdownWeb3 - 具有互動式 AI 聊天功能的 Jekyll 網站

## 專案概覽
一個基於 Jekyll 建構的網站，專注於數位身份、Web 3.0 和 AI 主題。為 GitHub Pages 部署而建，具有即時聊天介面和圖片上傳功能等自定義互動特性。

## 架構與核心組件

### 內容結構
- **主要頁面**: 位於根目錄資料夾（`DeZeta/`、`YouTuber/`）的自定義頁面，使用前置資料中的 `order` 進行導航排序
- **部落格文章**: `_posts/` 中的標準 Jekyll 文章，包含中文內容和技術見解
- **導航系統**: 使用前置資料中的 `page.order` 進行自定義排序，在 `_includes/header.html` 中排列

### 互動功能
網站實現了模組化的 JavaScript 架構，使用 ES6 模組：

```javascript
// 整個專案使用的模組模式
import displayMessage from './ds_chat_lib.js';  // 預設導出
import { debugLog } from './abc_def.js';        // 具名導出
```

#### 聊天系統 (`assets/js/ds_chat.js`)
- 與 Flask 後端通訊的即時 AI 聊天介面
- 透過 `.js-parameter` 元素上的 `data-*` 屬性進行 HTML 到 JS 的參數傳遞
- 雙環境支援：本地開發（`http://127.0.0.1:8080`）vs 正式環境（`https://aws.holymap.com.tw`）

#### 圖片上傳 (`assets/js/ds_upload_img.js`)
- 多檔案圖片上傳與 AI 識別功能
- 使用 FormData API 將檔案傳送到 `/api/identify` 端點

### 樣式架構
自定義 SCSS 模組導入到 `assets/main.scss`：
```scss
@import "minima";           // 基礎 Jekyll 主題
@import "_ds_chat";         // 聊天介面樣式
@import "_ds_upload_img";   // 上傳組件樣式
```

## 開發模式與慣例

### Jekyll 配置
- **GitHub Pages 設定**: 使用 `github-pages` gem 而非獨立的 Jekyll
- **Baseurl 配置**: 設為 `/MarkdownWeb3` 以適應 GitHub Pages 子路徑部署
- **主題**: Minima 2.5 搭配自定義覆寫

### JavaScript 模組系統
- **函式庫結構**: 分離的 `*_lib.js` 檔案用於可重用函數，主要 `*.js` 檔案用於初始化
- **環境切換**: 調試功能自動根據運行環境啟用（localhost、127.0.0.1 或 ?debug=true）
- **錯誤處理**: 一致的 fetch API 錯誤處理，提供用戶友好的錯誤訊息

### 內容慣例
- **中英雙語**: 雙語內容，技術術語使用英文
- **前置資料**: 使用 `order` 欄位進行導航排序，`permalink` 設定自定義 URL
- **註解**: 內聯 AI 工具歸屬註解（如 `// Copilot:`、`// DeepSeek:`）用於代碼生成追蹤

## 開發工作流程

### 本地開發
```bash
bundle exec jekyll serve
# 網站運行於 http://127.0.0.1:4000/MarkdownWeb3/
```

### 關鍵檔案關係
- `index.markdown` → 主頁面，包含聊天介面和模組匯入
- `_config.yml` → 關鍵：修改後需重啟 Jekyll 伺服器
- `assets/main.scss` → 所有自定義樣式的進入點
- `_includes/header.html` → 具有排序頁面的自定義導航

### API 整合
後端端點預期格式：
- **聊天**: POST 到 `/api/aigc`，JSON 格式 `{UserID, message}`
- **圖片**: POST 到 `/api/identify`，FormData 檔案上傳

## 專案特殊注意事項
- **行動 Line 整合**: 透過加好友小工具整合 Line 訊息應用
- **影片嵌入**: 使用 flexbox 的響應式 YouTube iframe 容器
- **Copilot 註解**: 保留 AI 生成歸屬註解以供參考
- **資源組織**: JavaScript 模組遵循 `{功能}_{類型}.js` 命名慣例

## 檔案結構慣例
```
assets/js/
├── abc_def.js          # 常數和配置
├── abc_lib.js          # 核心 API 通訊
├── ds_chat.js          # 聊天介面控制器
├── ds_chat_lib.js      # 聊天訊息顯示工具
├── ds_upload_img.js    # 上傳介面控制器
└── ds_upload_img_lib.js # 上傳工具

_sass/
├── _ds_chat.scss       # 聊天介面樣式
└── _ds_upload_img.scss # 上傳組件樣式
```

修改 JavaScript 時：同時更新控制器和函式庫檔案。新增功能時：遵循模組化模式，為可重用函數建立分離的函式庫檔案。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/hush-albert)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/hush-albert)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
