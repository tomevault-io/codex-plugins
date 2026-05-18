# Shell Module — 项目指南

轻量级嵌入式 CLI 模块，面向 ARM Cortex-M 裸机环境，通过 UART 提供交互式命令行。

## 架构

| 文件                            | 职责                                                                                               |
| ------------------------------- | -------------------------------------------------------------------------------------------------- |
| `shell.h` / `shell.c`           | 核心实现：命令解析、行编辑、ESC序列、历史记录、Tab补全、变量读写、权限认证、透传模式、日志尾行输出 |
| `shell_cfg.h`                   | 功能开关与参数配置（缓冲区大小、功能裁剪宏）                                                       |
| `shell_port.h` / `shell_port.c` | 移植层：硬件 IO 适配、用户/命令/变量注册（示例基于 STM32 HAL + LWRB）                              |

**核心设计决策：**

- 命令通过 `SHELL_EXPORT_CMD` 宏 + 链接器 section 自动注册，无需集中式命令表
- 变量通过 `SHELL_EXPORT_VAR` 宏导出，支持运行时读写
- 支持 Keil(ARMCC)、GCC、IAR 三种编译器的 section 语法
- `shell_t` 控制块静态分配，零动态内存

## 代码规范

- 使用 `stdint.h` 固定宽度类型，禁止裸 `int`/`short`/`long`（命令回调签名 `int func(int argc, char* argv[])` 除外，这是惯例接口）
- 禁止 `malloc`/`free`，所有内存静态分配
- 硬件寄存器访问使用 `volatile`
- 静态局部变量前缀 `s_`，全局变量前缀 `g_`
- 中文注释
- 字符串常量集中定义为 `STR_xxx` 宏，便于多语言和自定义

## 移植要点

适配新硬件时只需修改 `shell_port.c`：

1. 实现 `write` 回调（发送数据到串口）
2. 实现 `read` 回调（从缓冲区读取接收数据），或使用内置环形缓冲区 + `shell_rx_push()`
3. 配置用户表（若启用 `SHELL_USING_AUTH`）
4. 在主循环中周期调用 `shell_task()`

## 功能裁剪

通过 `shell_cfg.h` 中的宏开关控制功能编译：

- `SHELL_USING_CMD_EXPORT` — 宏导出命令（需链接脚本）
- `SHELL_USING_VAR` — 变量读写
- `SHELL_USING_HISTORY` — 历史记录
- `SHELL_USING_COMPLETION` — Tab 补全
- `SHELL_USING_PASSTHROUGH` — 透传模式
- `SHELL_USING_AUTH` — 用户权限认证

## 构建与测试

主机端单元测试（需 GCC + Unity）：

```bash
cd test
make unity   # 首次：下载 Unity 框架
make         # 编译并运行全部测试
make clean   # 清理
```

测试通过直接 `#include "../shell.c"` 将实现内联编译，并通过宏覆盖 `SHELL_VAR_LIST` / `SHELL_VAR_COUNT` 来替代链接器 section 机制。

## 链接脚本

GCC 需要在链接脚本中添加 `.shell_cmd` 和 `.shell_var` 段；Keil scatter 文件需要添加 `SHELL_CMD` / `SHELL_VAR` 执行区域。具体格式见 `shell.h` 头部注释。

---
> Source: [causebefore/shell-module](https://github.com/causebefore/shell-module) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-05-18 -->
