# NanoBanana — Copilot 工作区指令

## 项目概况

- **类型**：AI 图像/视频生成平台
- **前端**：Vue 3 + Naive UI + Pinia + Vite（`frontend/`）
- **后端**：Go (Gin) + GORM + MySQL（`backend/`）
- **小程序**：微信小程序（`miniprogram/`）

## 设计规范

- 暗色电影质感主题，紫粉渐变品牌色 (`#667eea` → `#764ba2`)
- 强调色 `#a5b4fc`，背景 `#0a0a12` / `#0f0f1a`
- 玻璃态设计：`backdrop-filter: blur(24px) saturate(150%)`
- 字体 Inter，圆角 12–24px，`cubic-bezier(0.4, 0, 0.2, 1)` 过渡

## 开发指令

- 前端开发服务器：`cd frontend && node ./node_modules/vite/bin/vite.js --host`
- 前端构建：`cd frontend && npx vite build`
- 后端启动：`cd backend && go run main.go`
- API 代理：前端 dev server 自动代理 `/api` → `localhost:8092`

## 编码规范

- Vue 组件使用 `<script setup>` 组合式 API
- CSS 使用 `<style scoped>`，遵循 CSS 自定义属性设计系统（见 `style.css`）
- Go 后端遵循标准项目布局 (`internal/api`, `internal/db`, etc.)
- 保持沟通简洁，优先使用工具而非解释

---
> Source: [capybara-zy/xiaoye-ai](https://github.com/capybara-zy/xiaoye-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-04-20 -->
