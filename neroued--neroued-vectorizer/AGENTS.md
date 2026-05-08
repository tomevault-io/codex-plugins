
# 工具函数复用与实现结构

## 工具函数复用

- 禁止重复编写语义相同的工具函数。
- 新增工具函数前，先检索项目内是否已有可复用实现。
- 跨模块复用的函数应放入公共头文件（如 `src/detail/`）。
- 仅当前文件使用的工具函数应保留为匿名命名空间局部实现。

## .cpp 文件结构

- 避免在 .cpp 内做非必要前向声明；优先通过合理的函数定义顺序解决依赖。
- 实现文件应按"工具函数 → 核心流程 → 对外接口"组织。

## 命名空间约定

- 公共 API 使用 `neroued::vectorizer` 命名空间。
- 内部实现使用 `neroued::vectorizer::detail` 命名空间。
- 公共头文件位于 `include/neroued/vectorizer/`，内部头文件位于 `src/` 子目录。

---
> Source: [Neroued/neroued_vectorizer](https://github.com/Neroued/neroued_vectorizer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-04 -->
