# GitHub Copilot Instructions for RT-Thread / RT-Thread GitHub Copilot 指南

## Overview / 概述

RT-Thread is a real-time operating system (RTOS) for embedded devices. When working with RT-Thread code, please follow these guidelines to ensure high-quality contributions.

RT-Thread 是一个面向嵌入式设备的实时操作系统（RTOS）。在处理 RT-Thread 代码时，请遵循以下指南以确保高质量的贡献。

## Code Review Guidelines / 代码审查指南

### Language Requirements / 语言要求

When reviewing code, provide feedback in **both English and Chinese** to ensure accessibility for all contributors.

在审查代码时，请同时使用**英文和中文**提供反馈，以确保所有贡献者都能理解。

### Review Focus Areas / 审查重点领域

1. **Real-Time Performance / 实时性能**
   - Verify interrupt handling efficiency / 验证中断处理效率
2. **Memory Management / 内存管理**
   - Detect memory leaks / 检测内存泄漏
   - Verify proper memory allocation/deallocation / 验证正确的内存分配/释放
   - Check stack usage optimization / 检查栈使用优化
3. **Code Style / 代码风格**
   - Follow RT-Thread coding standards / 遵循 RT-Thread 编码标准
   - Maintain consistent naming conventions / 保持一致的命名约定
   - Ensure proper code comments (not documentation) / 确保适当的代码注释（而非文档）
4. **PR Title Naming Guidelines / PR 标题命名规范**
   - **Specify the module or keyword / 明确模块或关键字**：
     - 标题需明确指出涉及的具体模块、子系统或关键字，例如具体的 BSP（Board Support Package）或芯片厂商名称（如 STM32, ESP32, NXP 等）。
     - 示例：[STM32][I2C] Fix Kconfig parsing error 而非 fix:I2C——Kconfig修改。
   - **Clearly describe the content being repaired or modified / 清晰描述修复或更改内容**：
     - 标题需简洁清晰地描述修复的问题、添加的功能或修改的内容，避免模糊或过于简略的描述。
     - 示例：[STM32][SPI] Fix buffer overflow in SPI driver 而非 SPI bug fix。
   - **Format Recommendations / 格式建议**：
     - 推荐使用 [模块/厂商][子系统] 具体描述 的格式，确保标题结构化且信息完整。
     - 使用英文描述问题（除非项目明确要求使用其他语言），以提高国际化可读性。
     - 示例：[NXP][UART] Add timeout handling for UART receive。
   - **Issues to Avoid / 避免的问题**：
     - 不要使用模糊的术语，如“修复问题”或“代码优化”，需具体说明问题或优化的内容。
     - 避免使用不规范的符号（如 ——），建议使用标准英文字符（如 - 或 :）。
     - 不要省略关键上下文信息，如 BSP 或芯片厂商。

### Review Comment Format / 审查评论格式

When providing review comments, use the following format:

提供审查评论时，请使用以下格式：

```
[Category/类别]: Brief description / 简要描述

English: Detailed explanation of the issue and suggested improvement.
中文：问题的详细说明和改进建议。

Example/示例:
```c
// Your code example here / 你的代码示例
```
```
### Common Issues to Check / 常见问题检查

1. **Resource Management / 资源管理**
   - Unclosed file handles / 未关闭的文件句柄
   - Unreleased semaphores / 未释放的信号量
   - Memory not freed after malloc / malloc 后未释放内存

2. **Error Handling / 错误处理**
   - Missing error checks / 缺少错误检查
   - Improper error propagation / 不当的错误传播
   - Silent failures / 静默失败

3. **Performance Concerns / 性能问题**
   - Unnecessary polling / 不必要的轮询
   - Inefficient algorithms in ISRs / ISR 中的低效算法
   - Excessive context switching / 过度的上下文切换

### Severity Levels / 严重程度级别

- **🔴 Critical/严重**: Issues that may cause system crashes or data corruption / 可能导致系统崩溃或数据损坏的问题
- **🟠 Major/主要**: Significant bugs or performance issues / 重大错误或性能问题
- **🟡 Minor/次要**: Code style or minor optimization opportunities / 代码风格或次要优化机会
- **🟢 Suggestion/建议**: Best practices or enhancement ideas / 最佳实践或增强建议

## RT-Thread Specific Guidelines / RT-Thread 特定指南

### Kernel Components / 内核组件

When reviewing kernel-related code:
审查内核相关代码时：

- Verify rt_thread structure usage / 验证 rt_thread 结构使用

### Device Drivers / 设备驱动

For device driver reviews:
对于设备驱动审查：

- Ensure proper device registration / 确保正确的设备注册
- Verify I/O operation handling / 验证 I/O 操作处理

### Network Stack / 网络协议栈

When reviewing network code:
审查网络代码时：

- Validate SAL (Socket Abstraction Layer) usage / 验证 SAL（套接字抽象层）使用
- Check protocol implementations / 检查协议实现
- Ensure proper buffer management / 确保正确的缓冲区管理

## Coding Standards / 编码标准

### Object-Oriented Design in C / C语言面向对象设计

1. **Inheritance Pattern / 继承模式**
   - First member should be base struct / 第一个成员希望是基类结构体
   - Use pointer casting for type conversion / 通过指针强制转换实现类型转换

2. **Polymorphism via ops / 通过ops实现多态**
   - Define ops struct with function pointers / 定义包含函数指针的ops结构体
   - Share single ops table across instances / 多个实例共享同一ops表

### Naming Conventions / 命名规范

- **Structures / 结构体**: `rt_[name]`
- **Public Functions / 公开函数**: `rt_[class]_[action]`
- **Static Functions / 静态函数**: `_[class]_[action]`
- **Hardware Functions / 硬件函数**: `rt_hw_`
- **Macros / 宏定义**: UPPERCASE (except for local function/variable macros)
- **Error Codes / 错误码**: `RT_` + POSIX error code, `RT_EOK` for success

### Object Lifecycle / 对象生命周期

- Provide dual APIs / 提供双模式API:
  - `init/detach` for static objects / 用于静态对象
  - `create/delete` for dynamic objects / 用于动态对象
- Use reference counting / 使用引用计数
- Return unified error codes / 返回统一错误码

### Code Format / 代码格式

- 4 spaces indentation, no tabs / 4空格缩进，不使用tab
- Braces on separate lines / 大括号独占一行
- Align parameters on line breaks / 参数换行时对齐

## Documentation Standards / 文档标准

### Language and Format / 语言和格式

- Use English for code comments / 所有代码注释使用英文
- Markdown format for documentation / 文档使用Markdown格式
- Prefer Mermaid for diagrams, or PlantUML (hide footbox in sequence diagrams) / 优先使用Mermaid绘图，或PlantUML（时序图隐藏footbox）

### Document Structure / 文档结构

1. **Main Level / 主干层**: Overall overview / 整体概述
2. **Branch Level / 分支层**: Module introduction / 子模块介绍
3. **Node Level / 节点层**: Detailed knowledge points / 知识点详解

### Documentation Principles / 文档原则

- Keep structure flat / 保持扁平结构
- Modular organization / 模块化组织
- Clear and concise content / 内容简洁直接
- Complete executable examples / 完整可执行示例

## Best Practices / 最佳实践

1. **Always consider embedded constraints** / **始终考虑嵌入式约束**
   - Limited RAM and ROM / 有限的 RAM 和 ROM
   - Power consumption / 功耗
   - Real-time requirements / 实时要求
   - Prefer static allocation / 优先静态分配
   - Use memory alignment / 使用内存对齐

2. **Verify on real hardware or at least QEMU** / **尽可能在真实硬件上验证，或至少在QEMU上验证**
   - Test on actual hardware when available / 有条件时在真实硬件上测试
   - Use QEMU simulation as minimum verification / 至少使用QEMU仿真进行验证
   - Consider various BSP configurations / 考虑各种 BSP 配置

3. **Document hardware dependencies** / **记录硬件依赖**
   - Specify required peripherals / 指定所需外设
   - Note timing constraints / 注意时序约束
   - List supported MCU/MPU families / 列出支持的 MCU/MPU 系列

4. **Code Optimization / 代码优化**
   - Use `rt_inline` for simple functions / 简单函数使用rt_inline
   - Avoid deep nesting / 避免深层嵌套
   - Single responsibility per function / 函数功能单一
   - Minimize code duplication / 减少代码重复

## Contributing / 贡献

When suggesting improvements:
提出改进建议时：

1. Provide clear, actionable feedback / 提供清晰、可操作的反馈
2. Include code examples when possible / 尽可能包含代码示例
3. Reference RT-Thread documentation / 引用 RT-Thread 文档
4. Consider backward compatibility / 考虑向后兼容性

## References / 参考资料

- [RT-Thread Documentation](https://www.rt-thread.io/document/site/)
- [RT-Thread Coding Style Guide](https://github.com/RT-Thread/rt-thread/blob/master/documentation/coding_style_en.md)
- [RT-Thread 文档中心](https://www.rt-thread.org/document/site/)
- [RT-Thread 编码规范](https://github.com/RT-Thread/rt-thread/blob/master/documentation/coding_style_cn.md)
```

---
> Source: [nextpilot/nextpilot-flight-control](https://github.com/nextpilot/nextpilot-flight-control) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:agents_md:2026-07-26 -->
