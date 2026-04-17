# AGENTS.md — 全局工作指令

## 项目概述

这是一个**智慧学习平台**（smart-learn-codex），前后端分离：
- **前端**：`app/` — React 19 + TypeScript + Vite 7 + TailwindCSS 3.4 + shadcn/ui + zustand 5 + framer-motion + recharts + react-router-dom 7 + axios + sonner
- **后端**：`backend/` — Express + Prisma (PostgreSQL) + JWT + Zod + MinIO 文件存储
- **数据库模型**：`backend/prisma/schema.prisma`
- **静态 UI 稿**：`static-ui/` — 每个子目录含 `code.html`（设计师 HTML 稿）和 `screen.png`（视觉参考）

## 当前任务

将 `static-ui/` 下的新 UI 逐页接入现有系统。**不仅仅是换皮——如果新 UI 包含了现有代码里不存在的功能（按钮、交互、数据展示），你必须完整实现这些功能的全栈逻辑。**

## ⛔ 绝对禁止（违反任何一条即视为失败）

1. **禁止 mock 数据** — 任何出现在 UI 上的数据都必须来自真实的 API 调用。不允许在前端写死 `const data = [{ ... }]` 之类的硬编码数组/对象来假装数据。
2. **禁止"功能开发中"** — 不允许出现 `toast("功能正在开发中")` 或 `alert("Coming soon")` 或 `console.log("TODO")` 之类的占位。如果一个按钮出现在 UI 上，它就必须能正常工作。
3. **禁止 TODO/FIXME 注释后就不管** — 不允许留下 `// TODO: implement this` 然后不实现。如果你写了 TODO，你必须在同一次提交中完成它。
4. **禁止空函数体** — `const handleXxx = () => {}` 这种空的事件处理器不允许出现。
5. **禁止删除现有功能** — 现有组件中已经在工作的功能不允许因为换 UI 而被删掉。

## ✅ 允许且鼓励的操作

1. **可以引入新的 npm 包** — 如果新 UI 需要日期选择器、拖拽、动画等，可以 `npm install` 新包。在 `app/` 或 `backend/` 目录执行。
2. **可以修改后端代码** — 如果新 UI 需要新接口或对现有接口的扩展，直接修改 `backend/src/routes/` 下的路由文件。
3. **可以修改 Prisma schema** — 如果需要新的字段或模型，修改 `backend/prisma/schema.prisma` 并生成迁移。
4. **可以新增 zustand store 方法** — 如果新 UI 需要新的数据操作，在现有 store 文件中添加方法或创建新的 store 文件。
5. **可以新增 useState / useEffect** — 如果新 UI 有新的交互逻辑（如新的筛选器、弹窗、表单），添加相应的 state 和 effect。
6. **可以修改布局组件** — `app/src/components/layout/MainLayout.tsx` 和 `AuthLayout.tsx` 可以根据新 UI 调整。
7. **可以新增公共组件** — 如果多个页面需要复用的 UI 元素，创建到 `app/src/components/` 下。

## 🎯 核心原则

**每个按钮必须有真实功能。** 判断标准很简单：
- 用户点击这个按钮 → 调用 API → 后端处理 → 返回结果 → 前端展示反馈
- 如果静态 UI 里有一个按钮，但现有系统没有这个功能，你需要：
  1. 在后端添加对应的 API 路由
  2. 在前端 store 添加对应的方法
  3. 在组件中绑定完整的调用链

**所有展示的数据必须来自 API。** 判断标准：
- 数字、列表、图表数据 → 来自 store → 来自 API → 来自数据库
- 如果静态 UI 展示了一个"本月活跃度排名"但现有 API 没有这个数据，你需要在后端添加这个聚合查询接口

## 工作流程

对每个任务的每个文件：

### Step 1: 分析新 UI
读取 `static-ui/《目录名》/code.html`，对比现有组件，列出：
- 纯视觉变化（布局、配色、间距）→ 只改 JSX + TailwindCSS
- 新增交互（按钮、弹窗、切换）→ 添加 state + handler
- 新增数据展示 → 检查是否有现成 API，没有就先实现 API

### Step 2: 后端优先（如需要）
如果需要新接口：
1. 修改 `backend/prisma/schema.prisma`（如需新字段/模型）
2. 在 `backend/src/routes/` 中添加或扩展路由
3. 确保路由在 `backend/src/index.ts` 中已注册

### Step 3: Store 扩展（如需要）
在 `app/src/stores/` 对应的 store 中添加新方法，或创建新 store。

### Step 4: 组件重写
- 保留所有现有逻辑绑定（hooks、handlers、条件渲染、列表渲染）
- 重写 JSX 匹配新 UI 视觉
- 用 shadcn/ui 组件替代原生 HTML
- 用 lucide-react 替代 SVG 图标
- 新增必要的 state 和 handler 来支持新交互

### Step 5: 验证
```bash
cd app && npx tsc --noEmit
cd backend && npx tsc --noEmit
```

## 技术栈速查

| 层级 | 技术 |
|---|---|
| 前端框架 | React 19, TypeScript, Vite 7 |
| 样式 | TailwindCSS 3.4, cn() utility |
| UI 组件 | shadcn/ui (Radix UI), 文件在 `app/src/components/ui/` |
| 图标 | lucide-react |
| 动画 | framer-motion |
| 图表 | recharts (封装在 `app/src/components/charts/`) |
| 状态管理 | zustand 5 (`app/src/stores/`) |
| 路由 | react-router-dom 7 (定义在 `app/src/App.tsx`) |
| HTTP | axios (封装在 `app/src/lib/api.ts`) |
| 通知 | sonner (toast) |
| 后端 | Express, Prisma, PostgreSQL |
| 认证 | JWT (`backend/src/middleware/auth.ts`) |
| 文件存储 | MinIO (`backend/src/services/storage/`) |
| 校验 | zod (前后端均用) |

## static-ui ↔ React 组件映射

| static-ui 目录 | 目标文件 |
|---|---|
| 用户登录页面 | `app/src/pages/auth/LoginPage.tsx` + `app/src/components/layout/AuthLayout.tsx` |
| 用户注册页面 | `app/src/pages/auth/RegisterPage.tsx` |
| 学习行为与积分仪表盘 | `app/src/pages/student/StudentDashboard.tsx` + `StudentAnalyticsPage.tsx` |
| ai_助手界面 | `app/src/pages/student/AIAssistantPage.tsx` |
| 课程主页 | `app/src/pages/student/CourseDetailPage.tsx` |
| 线上资源库 | `app/src/pages/student/ResourceLibraryPage.tsx` + `ResourceDetailPage.tsx` |
| 课程思政案例库_1 + _2 | `app/src/pages/student/CaseLibraryPage.tsx` |
| 学生上传作业页面_1 + _2 + 学生加入班级页面 | `app/src/pages/student/StudentHomeworkPage.tsx` |
| 同行互评中心 | `app/src/pages/student/PeerReviewPage.tsx` |
| 个人设置 | `app/src/pages/student/SettingsPage.tsx` |
| 教师创建班级页面 | `app/src/pages/teacher/ClassManagementPage.tsx` |
| 教师发布作业页面_1 + _2 | `app/src/pages/teacher/HomeworkManagementPage.tsx` |
| 教师作业批改详情 + 项目作业综合批改页 | `app/src/pages/teacher/HomeworkManagementPage.tsx` |
| 教师管理后台_-_作业与干预 | `app/src/pages/teacher/InterventionConsolePage.tsx` + `TeacherDashboard.tsx` |
| 教师管理后台_-_学生行为数据 | `app/src/pages/teacher/BehaviorAnalysisPage.tsx` + `AnalyticsPage.tsx` |
| 平时表现管理中心 | `app/src/pages/teacher/ClassPerformancePage.tsx` |
| 作业动态组队中心 | `app/src/pages/teacher/HomeworkManagementPage.tsx` |

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/caixiaoshun) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-17 -->
