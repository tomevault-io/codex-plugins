# Copilot 工作区指南

## 项目概述

Neo4j 图数据库可视化对比工具，使用 **Vue 3 + Vite + Pinia + TDesign**，对比 G6、Neovis.js、HTTP 透传三种渲染方案的性能。

## 角色与目标

- 你是一名资深的全栈开发专家，熟练使用使用各种主流编程语言及其技术栈。
- 你的核心任务是，根据需求开发完整、生产级别的全栈功能包或插件。你必须严格遵循项目内现有的设计模式，确保你生成的每一部分代码都能正常运行。
- 你的主要职责包括：
  - 理解并遵循项目的架构模式和编码规范。
  - 根据需求设计和实现功能模块。
  - 进行前端用户界面的设计与开发，确保良好的用户体验。
  - 编写高质量、可维护的代码，不要重构无关代码，并附带必要的注释和文档。
- 使用中文进行交流和代码注释。

## Agent工作注意事项

- 本项目目的是使用第三方图表库快速实现 Neo4j 数据的可视化对比，重点在于性能比较和功能展示。在编写代码前，请使用 Context7 mcp 获取正确的库用法和最佳实践，避免不必要的重构和过度设计。

## 构建与运行

```bash
npm install        # 安装依赖
npm run dev        # 开发服务器，默认 http://localhost:5173（自动打开浏览器）
npm run build      # 生产构建
npm run preview    # 预览构建产物
```

无测试脚本。无需代理配置，前端直连 Neo4j Bolt 协议。

## 代码风格

- **SFC 结构**：`<template>` → `<script setup>` → `<style scoped>`，禁用 Options API 和 TypeScript
- **Composition API**：原始值用 `ref`，对象用 `reactive`；从 store 解构必须用 `storeToRefs`，禁止裸解构
- **样式**：全部 `scoped`，TDesign CSS token（`var(--td-brand-color)` 等），跨组件穿透用 `:deep()`
- 示例参考：[src/components/G6Renderer.vue](../src/components/G6Renderer.vue)、[src/stores/neo4j.js](../src/stores/neo4j.js)

## 架构

**路由**（[src/router/index.js](../src/router/index.js)）：全部懒加载，`/demo/g6`、`/demo/neovis`、`/demo/http` 三条演示路由。

**Store**（Setup Store 风格，非 Options）：
- `useNeo4jStore`：Neo4j 连接、查询的核心 store，含 sessionStorage 持久化（密码不持久化）
- `useAppStore`：占位用，暂未扩展

**数据转换链**（[src/utils/transform.js](../src/utils/transform.js)）：
```
Neo4j Records → transformRecords() → { nodes[], edges[] }
  → toG6Data()          # G6 格式
  → toVisNetworkData()  # vis-network DataSet 格式
```
HTTP Demo 使用独立的 `transformHttpRecords`，兼容 Go 驱动的大写字段名（`ElementId`、`Labels`、`Props`）和 `{High, Low}` int64 结构。

**渲染器组件**：
- `G6Renderer`：纯展示，`graphData` prop，emit `render-start/render-end`，内置 12 色调色板
- `NeovisRenderer`：双模式，`config`（neovis 原生）或 `graphData`（vis-network 手动），`defineExpose` 暴露 API
- `PerfPanel`：纯展示，接收 `usePerfMetrics` 返回的 `metrics` 对象

## 性能基准

[src/composables/usePerfMetrics.js](../src/composables/usePerfMetrics.js) 通过 `performance.now()` + `requestAnimationFrame` + `performance.memory` 采集渲染时间、avg FPS、内存增量。事件流：`render-start` → `startMeasure(nodeCount, edgeCount)` → `render-end` → `endMeasure()`。

## OpenSpec 工作流

变更记录位于 [openspec/changes/](../openspec/changes/)，每个变更包含 `proposal.md`、`design.md`、`tasks.md` 和 `specs/`。`archive/` 存放已完成变更。当前进行中：`render-perf-compare`（tasks 1–8.2 已完成，8.3 手动验证待做）。

## 关键依赖说明

- `@antv/g6` v5：API 与 v4 不兼容，`Graph` 实例创建方式已变更
- `neovis.js` v2：内部依赖 `vis-network` / `vis-data`，需通过 `destroyViz()` 清理
- Vite 配置中 `optimizeDeps.esbuildOptions.sourcemap: false` 用于加速大依赖（G6、neovis）预构建

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/luluan23)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/luluan23)
<!-- tomevault:4.0:agents_md:2026-04-09 -->
