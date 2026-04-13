<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# 個人投資決策平台 - Copilot 指令

## 專案概述
這是一個以「每日紀律」為核心的投資決策管理平台，幫助投資人培養良好的投資習慣和決策紀律。

## 專案目標
- 每天 3-5 分鐘內快速記錄投資決策
- 追蹤短、中、長期投資計畫執行情況
- 管理市場事件和風險提醒
- 培養投資紀律，減少情緒化操作

## 技術棧
- **框架**: Next.js 16+ (App Router)
- **語言**: TypeScript
- **樣式**: Tailwind CSS 4
- **部署**: Azure Static Web Apps
- **導出模式**: 靜態導出 (Static Export)
- **狀態管理**: React useState (未來考慮 Zustand)
- **資料儲存**: LocalStorage (Phase 1) → 後端 API (Phase 2+)

## 開發階段
當前處於 **Phase 1: 每日決策與紀律（MVP）**

### Phase 0-1 已完成
- ✅ 主頁 Dashboard UI
- ✅ 投資計畫管理（短/中/長期）
- ✅ 紀錄行事曆
- ✅ 市場事件追蹤
- ✅ 風險提醒系統
- ✅ 所有核心元件

### Phase 1 待開發
- ⏳ 每日紀錄填寫表單
- ⏳ 紀錄詳細查看頁面
- ⏳ 資料持久化（LocalStorage）
- ⏳ 提醒功能

## 開發指南

### 1. 類型定義
- 所有投資決策相關類型在 `src/types/investment.ts`
- 核心類型：`DailyRecord`, `InvestmentPlan`, `MarketEvent`, `RiskReminder`
- 舊版投資組合類型在 `src/types/portfolio.ts`（保留備用）

### 2. 模擬資料
- 投資決策資料在 `src/lib/investmentMockData.ts`
- 舊版資料在 `src/lib/mockData.ts`（保留備用）

### 3. 元件開發
- 使用函數式元件和 TypeScript
- 需要互動的元件使用 `'use client'`
- 優先使用 Tailwind CSS
- 保持元件簡潔，單一職責

### 4. 顏色系統
```typescript
// 計畫類型
long: 藍色系 (bg-blue-50, text-blue-700)
mid: 綠色系 (bg-green-50, text-green-700)
short: 橘色系 (bg-orange-50, text-orange-700)

// 重要性/風險等級
high/critical: 紅色系
medium/warning: 黃色系
low/info: 灰色/藍色系
```

### 5. 設計原則
- **快速操作**: 每個功能都要在 3-5 分鐘內完成
- **視覺優先**: 用顏色和圖示傳達資訊，減少閱讀
- **紀律強化**: 每天強制檢視計畫和風險
- **漸進式**: 從簡單開始，不做過早優化

## 程式碼風格
- 使用 TypeScript 嚴格模式
- 優先使用 interface 定義類型
- 函數命名清晰（handle, on, get, set 等前綴）
- 元件檔名使用 PascalCase
- 遵循 ESLint 規則

## 資料處理
- Phase 1 使用 LocalStorage
- 資料格式統一用 JSON
- 日期使用 Date 物件，儲存時轉 ISO string
- 預留未來 API 整合的介面

## 重要文件
- **PROJECT_DESIGN.md**: 完整設計文件和 Roadmap
- **SCREENS_COMPLETED.md**: 當前畫面完成度總結
- **DEPLOYMENT.md**: Azure 部署指南

## 下一步開發優先順序
1. 實作每日紀錄填寫表單
2. 實作 LocalStorage 儲存
3. 實作紀錄查看和編輯
4. 加入瀏覽器提醒功能
5. 實作資料匯出（JSON/CSV）

## 注意事項
- 確保所有功能與靜態導出相容（不使用 SSR）
- 投資資料是敏感資訊，考慮加密儲存
- 保持快速反應，避免複雜的計算
- 行動裝置友善（響應式設計）

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/tony0935929660)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/tony0935929660)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
