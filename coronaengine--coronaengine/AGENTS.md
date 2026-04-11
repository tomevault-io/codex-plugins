# CoronaEngine AI 编程指南

本文档为 AI 代理提供了在 CoronaEngine 代码库中高效工作的核心知识。

## 核心架构

CoronaEngine 是一个模块化、多线程、数据驱动的 C++ 游戏引擎。

- **中央协调器**: `Engine` 类 (`include/corona/engine.h`) 负责管理应用程序的生命周期（初始化、运行、关闭）和系统注册。
- **服务中心**: 引擎建立在 `CoronaFramework` 之上，使用其 `KernelContext` 作为提供日志、事件总线和系统管理的中央服务中心。
- **系统 (Systems)**: 功能被封装在独立的、继承自 `Kernel::SystemBase` 的“系统”中（位于 `src/systems/`）。每个系统（如 `DisplaySystem`, `MechanicsSystem`）都在其专用线程中运行，并通过覆盖 `get_priority()` 来定义其初始化顺序。
- **数据驱动与事件通信**: 系统之间是解耦的，主要通过 `KernelContext` 提供的全局 `EventBus` 进行通信。事件在 `include/corona/events/` 中定义。

## 开发工作流

### 构建项目

项目使用基于预设的 CMake 工作流。构建文件生成在 `build/` 目录中。

1.  **配置 (Windows, MSVC, Ninja)**:
    ```powershell
    cmake --preset ninja-msvc
    ```

2.  **构建 (Debug 模式)**:
    ```powershell
    cmake --build --preset msvc-debug
    ```

可执行文件和库位于 `build/` 目录下的相应目标文件夹中。

### 添加一个新系统

这是最常见的开发任务：

1.  **创建头文件**: 在 `include/corona/systems/` 中创建一个新的头文件，使其继承自 `Kernel::SystemBase` 并覆盖必要的方法（`get_name`, `get_priority`, `initialize`, `update`, `shutdown`）。
    ```cpp
  // include/corona/systems/my_new_system.h
    #pragma once
    #include <corona/kernel/system/system_base.h>
    
    namespace Corona::Systems {
      class MyNewSystem : public Kernel::SystemBase {
      public:
        auto get_name() const -> std::string_view override { return "MyNewSystem"; }
        auto get_priority() const -> i16 override { return 88; } // 使用一个唯一的优先级
        void initialize(ISystemContext* ctx) override;
        void update() override;
        void shutdown() override;
      };
    }
    ```

2.  **创建实现**: 在 `src/systems/src/mynewsystem/` 目录下添加 `.cpp` 实现文件。

3.  **注册系统**: 在 `src/engine.cpp` 的 `Engine::register_systems()` 方法中包含头文件并注册你的新系统。
    ```cpp
    // 在 src/engine.cpp 中
    #include <corona/systems/my_new_system.h>
    // ...
    void Engine::register_systems() {
      // ...
      sys_mgr->register_system(std::make_shared<Systems::MyNewSystem>());
      logger->info("  - MyNewSystem registered (priority 88)");
    }
    ```

## 代码风格与约定

- **格式化**: 在提交代码前，运行根目录下的 `code-format.ps1` 脚本以确保代码风格一致。
- **命名**:
  - 类型（类、结构体）使用 `CamelCase`。
  - 函数和变量使用 `snake_case`。
- **CMake**: 构建逻辑是模块化的，位于 `misc/cmake/` 中。核心引擎逻辑在 `src/CMakeLists.txt` 中定义。

## 关键文件和目录

- `src/engine.cpp`: 引擎的入口点和系统注册。
- `include/corona/`: 核心引擎组件的公共 API。
- `src/systems/`: 所有功能模块（系统）的所在地。
- `examples/`: 演示如何使用引擎功能的示例项目。
- `CMakeLists.txt`: 根 CMake 文件，协调整个构建过程。
- `docs/`: 包含架构和开发者指南等详细文档。

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/CoronaEngine)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/CoronaEngine)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
