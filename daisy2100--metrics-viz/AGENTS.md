開發時，請參照以下規範做開發。

## 目前使用框架
1. angular 19
2. primeNG 19 
3. tailwind css 3.4

## 開發方式

1. 使用 primeNG 當作基礎元件
2. 當樣式無法符合需求時，使用tailwind css3.4
3. 避免使用自訂的CSS。

## Scss

1. 變數化處理參數的能力，便於後續切換樣式
2. 使用現代的 @use 語法取代 @import
3. 避免使用棄用的 Sass 函數，改用現代化語法：
   - 使用 `color.adjust()` 取代 `darken()` 和 `lighten()`
   - 使用 `@use 'sass:color'` 導入顏色模組
   - 建議使用預定義的顏色變數而非動態計算

## 顏色處理最佳實踐

- 在 `_variables.scss` 中預定義所需的顏色變體
- 使用 `darken-color()` 和 `lighten-color()` 輔助函數
- 避免在元件樣式中直接使用 Sass 顏色函數

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Daisy2100) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
