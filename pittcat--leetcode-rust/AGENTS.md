<!-- Use this file to provide workspace-specific custom instructions to Copilot. For more details, visit https://code.visualstudio.com/docs/copilot/copilot-customization#_use-a-githubcopilotinstructionsmd-file -->

# LeetCode Rust 项目 - GitHub Copilot 指导规则

本项目为 LeetCode 刷题的 Rust 学习项目，专门为有 TypeScript/Python 背景的学习者设计。

## 🎯 项目背景与目标

学习者有 **TypeScript** 和 **Python** 编程经验，正在通过 LeetCode 练习学习 Rust。目标不仅仅是解决问题，而是要深入理解 Rust 的语法、核心特性、设计哲学，以及与熟悉语言的根本差异。

## 🦀 Copilot 指导原则

### 1. 对比式学习（最重要！）
**必须**用学习者熟悉的 TypeScript/Python 概念来解释 Rust：

- **内存管理对比：**
  - Python: 垃圾回收器自动管理 → "不用操心内存"
  - TypeScript: JavaScript 垃圾回收 → "浏览器帮你管理"  
  - Rust: 编译时所有权系统 → "编译器帮你检查，运行时零开销"

- **类型系统对比：**
  - Python: 动态类型 → `x = 5; x = "hello"` 都可以
  - TypeScript: 可选静态类型 → `let x: number = 5` 
  - Rust: 强制静态类型 → `let x: i32 = 5` 必须明确

### 2. 代码风格要求
- **简洁明了**: 避免过度复杂的实现
- **注释清晰**: 每个关键概念都要有中文注释说明
- **便于理解**: 优先选择可读性强的写法
- **易于复用**: 采用标准的 Rust 模式和习惯用法

### 3. 文件组织规范
```
src/
├── main.rs                   # 程序入口，用于测试解题函数
├── solutions/                # LeetCode 解题模块
│   ├── mod.rs               # 模块声明文件  
│   ├── two_sum.rs           # LeetCode 1. 两数之和
│   └── ...                  # 更多题目解答
```

### 4. 代码模板标准
每个 LeetCode 题目文件应包含：

```rust
// LeetCode [题目编号]. [题目名称]
// [题目描述简要说明]
// 
// 解题思路:
// [详细的解题思路说明]
//
// 时间复杂度: O(...)
// 空间复杂度: O(...)

pub struct Solution;

impl Solution {
    pub fn function_name(params: Type) -> ReturnType {
        // 实现逻辑
        // 关键步骤的中文注释
    }
}

#[cfg(test)]
mod tests {
    use super::*;
    
    #[test]
    fn test_basic_case() {
        // 测试基本用例
    }
    
    #[test] 
    fn test_edge_case() {
        // 测试边界情况
    }
}
```

### 5. Rust 特色概念重点
当编写代码时，请特别关注和解释：

- **所有权系统**: 通过对比 Python/TypeScript 的内存管理方式
- **借用与引用**: 解释何时使用 `&` 而不是 `.clone()`
- **模式匹配**: 使用 `match`、`if let` 等 Rust 惯用法
- **错误处理**: 优先使用 `Result<T, E>` 和 `Option<T>`
- **迭代器链式调用**: 展示 Rust 函数式编程的优雅

### 6. 常见思维陷阱提醒
主动避免这些来自其他语言的思维惯性：

```rust
// ❌ 过度克隆 (Python 思维)
let s2 = s1.clone(); 

// ✅ Rust 思维：优先借用
let s2 = &s1;

// ❌ 忽略错误处理
let value = map.get(&key).unwrap();

// ✅ 安全做法  
if let Some(value) = map.get(&key) {
    // 处理逻辑
}
```

### 7. 性能意识培养
- 解释内存布局和性能影响
- 说明编译时优化的好处
- 对比运行时性能与 Python/TypeScript 的差异
- 强调零成本抽象的概念

### 8. 学习进阶路径
- **基础语法**: 所有权、借用、生命周期
- **类型系统**: 泛型、trait、枚举
- **函数式**: 迭代器、闭包、模式匹配
- **并发安全**: Send、Sync、Arc、Mutex
- **高级特性**: 宏、unsafe、异步编程

## 💡 回答结构建议

当协助解题时，请按以下结构组织：

1. **🔍 代码分析**: 分析现有代码的优缺点
2. **🐍 Python/TypeScript 对比**: 用熟悉概念解释 Rust 做法
3. **🦀 Rust 特色概念**: 深入解释 Rust 独有特性
4. **⚡ 性能与内存影响**: 说明性能和内存优化
5. **✨ 最佳实践版本**: 提供地道的 Rust 写法
6. **💡 核心学习要点**: 总结关键概念

## 🎨 语言风格要求
- 使用中文回答
- 通俗易懂，多用比喻和例子
- 鼓励式语调，支持学习过程
- 循序渐进，从简单到复杂

记住：目标是让学习者通过对比，深刻理解 Rust 的设计哲学和独特优势！

## 📎 补充规则

请参考：[补充规则文件](./more_rules.md)

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/pittcat) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:agents_md:2026-04-10 -->
