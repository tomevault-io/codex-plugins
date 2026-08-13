# Copilot Instructions for Modern Forum Project

## 项目架构与主要组件
- 本项目为纯前端论坛系统，核心文件包括：
  - `index.html`：主页面，展示帖子列表，分页与排序
  - `login.html`：用户登录页面，含验证码机制
  - `post.html`：帖子发布页面，支持分类、内容输入与格式化
  - `forum.js`：主要业务逻辑，包含帖子、评论、点赞、主题切换等功能
  - `repoLoader.ts`：可能用于数据加载或扩展（如需分析请补充说明）
- 样式采用 Tailwind CSS，图标使用 Font Awesome
- 用户状态与数据存储均通过 `localStorage` 实现，无后端 API，所有交互为前端模拟

## 关键开发模式与约定
- 业务逻辑集中在 `Forum` 对象（见 `forum.js`），所有异步操作采用统一的 `async/await` 风格，并有统一错误处理
- DOM 操作采用原生 JS，事件绑定在初始化阶段集中处理，避免重复绑定
- 所有用户操作（如点赞、评论）均有权限校验（需登录），并通过弹窗提示错误
- 帖子与评论内容渲染时强制 `escapeHtml`，防止 XSS
- 分页、搜索、主题切换等功能均有独立方法，便于扩展
- API 路径通过 `API` 常量统一管理，实际为前端模拟接口

## 测试与调试
- 测试文件位于 `__tests__/post.test.js`，如需扩展测试请保持同目录结构
- 直接在浏览器打开 HTML 文件即可调试，无需构建或启动服务
- 推荐使用浏览器开发者工具调试 JS 逻辑，关注控制台输出与弹窗提示

## 代码风格与扩展建议
- 所有新功能建议以 `Forum` 对象方法扩展，保持异步风格和统一错误处理
- 新页面请参考现有 HTML 文件结构与样式约定
- 如需集成第三方库，优先通过 CDN 引入，保持项目纯前端特性

## 示例：扩展点赞功能
```js
// 在 Forum 对象中添加新方法
async likePost(postId) {
  if (!this.currentUser || !this.currentUser.id) {
    alert('请先登录');
    return;
  }
  // ...异步处理与错误提示...
}
```

## 参考文件
- `forum.js`：所有核心业务逻辑
- `index.html`、`login.html`、`post.html`：页面结构与交互入口
- `README.md`：项目背景与功能说明

---
如有特殊约定或未覆盖的开发流程，请补充说明以便完善本说明。

---
> Source: [qiujing-git/luntan0826](https://github.com/qiujing-git/luntan0826) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-08-13 -->
