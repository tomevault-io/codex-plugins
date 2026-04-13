# GitHub Copilot 全局指令（项目专用）

当生成代码时，请遵循以下技术栈：

## 前端
- 框架：vue@3.5.24（Composition API + <script setup>）
- UI：tailwindcss@4.1.17
- 构建工具：Vite
- 风格：组件拆分清晰，使用响应式布局，代码简洁可维护

## 后端
- 框架：FastAPI 0.115.6
- 数据结构：Pydantic BaseModel
- 路由采用 APIRouter
- 返回 JSON 格式数据
- 遵守 RESTful API 设计

## 项目约定
- 前后端分离
- 所有新代码必须符合上述技术栈
- 自动生成示例时必须使用上述框架和语法

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/h617265630)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/h617265630)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
