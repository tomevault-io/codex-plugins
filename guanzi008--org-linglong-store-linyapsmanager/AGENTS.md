# Copilot Instructions

本文件为 GitHub Copilot 的仓库级说明，帮助 Copilot 和贡献者在本项目中生成更贴合的代码。请严格遵循下列约定与约束。

## 重点（极其重要）
在接到用户的任务的时候，先不要着急开始修改代码，要先分析需求，分析代码，列举解决方案，
详细的向用户说明你的思路，和你打算如何实现这个需求。
**只有在用户确认你的方案后，才开始动手写代码。**

## 概述
这个项目是一个 D-Bus 转发命令行输出的项目，目的是为了让玲珑应用商店在容器中能够调用外部的命令行工具（如 `ll-cli`, `killall` 等）。

**核心设计思想：**
- 服务端暴露单一的 `ExecuteCommand(command, args)` 方法
- 客户端通过**程序名自动识别**要执行的命令（符号链接）
- 所有命令统一使用流式输出

## 架构

### 服务端 (Server): `cmd/server/main.go`
- 在 `/org/linglong_store/LinyapsManager` 暴露 D-Bus 接口
- **单一方法**: `ExecuteCommand(command string, args []string) -> operationID`
- 使用白名单验证允许执行的命令
- 管理沙箱应用的环境变量和代理套接字

### 客户端 (Client): `cmd/client/main.go`
- 客户端二进制通过**符号链接**伪装成不同命令
- 自动识别调用时的程序名，作为要执行的命令
- 用户体验：`ll-cli install app` 感觉就像直接调用 `ll-cli`

### 白名单模块: `internal/cmdwhitelist/`
- `config.go`: 定义允许执行的程序列表和规则
- `validator.go`: 验证命令和参数的合法性

### 流式传输 (Streaming): `internal/streaming/`
- 使用 `operationID` 通过 D-Bus 信号流式传输命令输出
- 信号: `Output(operationID, data, isStderr)`, `Complete(operationID, exitCode, errorMsg)`

## 关键组件与模式

### 命令白名单
```go
// internal/cmdwhitelist/config.go
var AllowedCommands = map[string]CommandConfig{
    "ll-cli":  {Program: "ll-cli", NeedsEnv: true, AllowedSubcmds: [...]},
    "killall": {Program: "/usr/bin/killall", RequireArgs: true},
    "kill":    {Program: "/usr/bin/kill", RequireArgs: true},
    "pkexec":  {Program: "/usr/bin/pkexec", RequireArgs: true},
}
```

### 客户端符号链接
```bash
# 构建后的目录结构
build/
├── linyapsctl              # 客户端主程序
├── linyaps-dbus-server     # 服务端
├── ll-cli -> linyapsctl
├── killall -> linyapsctl
├── kill -> linyapsctl
└── pkexec -> linyapsctl
```

### D-Bus 通信
- **库:** `github.com/godbus/dbus/v5`
- **常量:** `internal/dbusconsts/consts.go`
- **连接:** `internal/dbusutil` 处理连接逻辑

### 流式协议
1. **客户端:** 调用 `ExecuteCommand(command, args)`，获取 `operationID`
2. **服务端:** 后台 goroutine 发送 `Output` 信号，最后发送 `Complete` 信号
3. **客户端:** 使用 `streaming.Receiver` 接收信号并输出

## 开发工作流

### 构建
```bash
# 使用 Makefile（推荐）
make

# 或手动构建
go build -o build/linyaps-dbus-server ./cmd/server
go build -o build/linyapsctl ./cmd/client
cd build && ln -s linyapsctl ll-cli
```

### 运行
```bash
# 启动服务端
./build/linyaps-dbus-server

# 使用客户端（通过符号链接）
./build/ll-cli list
./build/ll-cli install com.example.app
./build/killall firefox
```

### 测试
```bash
# 运行单元测试
go test ./...

# 测试白名单验证
go test ./internal/cmdwhitelist/...
```

## 添加新命令

1. 在 `internal/cmdwhitelist/config.go` 的 `AllowedCommands` 中添加配置
2. 在 `Makefile` 的 `SYMLINKS` 变量中添加命令名
3. 重新构建

```go
// 示例：添加 systemctl 命令
"systemctl": {
    Program:        "/usr/bin/systemctl",
    AllowedSubcmds: []string{"status", "start", "stop"},
    RequireArgs:    true,
},
```

## 常见任务

- **修改白名单规则:** 编辑 `internal/cmdwhitelist/config.go`
- **添加环境变量注入:** 在命令配置中设置 `NeedsEnv: true`
- **阻止危险参数:** 在配置的 `BlockedArgs` 中添加

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/guanzi008)
> Context snippets also available to append to your CLAUDE.md, GEMINI.md, and copilot-instructions.md — [download at TomeVault](https://tomevault.io/claim/guanzi008)
<!-- tomevault:4.0:agents_md:2026-04-08 -->
