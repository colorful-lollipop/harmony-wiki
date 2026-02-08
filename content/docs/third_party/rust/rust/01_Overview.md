# 原始库简介

## Rust 语言概述

**Rust** 是一个静态强类型语言，由 Mozilla 研究院开发并于 2010 年首次公开发布。其核心设计理念是提供：

- **内存安全** - 无垃圾回收器的内存安全保证
- **高性能** - 零成本抽象，接近 C/C++ 的性能
- **并发安全** - 类型系统防止数据竞争
- **现代化工具链** - Cargo 包管理器、clippy linter、rustfmt 格式化工具

## 版本信息

| 项目 | 值 |
|------|-----|
| **当前版本** | Rust 1.72.0 (基于上游 1.72.0) |
| **上游版本** | Rust 1.72.0 (2023-08-24 发布) |
| **上游地址** | https://github.com/rust-lang/rust |
| **许可证** | Apache-2.0 / MIT |

## 核心组件

Rust 工具链包含以下核心组件：

### 编译器 (`rustc`)

Rust 编译器是一个复杂的多阶段编译器，包含：

- **前端** - 词法分析 (`rustc_lexer`)、语法解析 (`rustc_parse`)、语义分析 (`rustc_hir_*`)
- **中端** - 类型检查 (`rustc_type_ir`)、借用检查 (`rustc_borrowck`)、MIR 生成 (`rustc_mir_*`)
- **后端** - 代码生成 (LLVM/Cranelift/GCC)、汇编与链接

### 标准库

| 库 | 描述 |
|----|------|
| `core` | 无 `std` 依赖的核心库，提供 `no_std` 支持 |
| `alloc` | 堆分配器 (Vec、String、Box 等)，依赖 `core` |
| `std` | 完整标准库，包含 I/O、网络、线程等 |
| `proc_macro` | 过程宏支持 |
| `portable-simd` | 便携式 SIMD 操作 |
| `test` | 测试框架 |

### 工具链

| 工具 | 用途 |
|------|------|
| `cargo` | 包管理器与构建系统 |
| `clippy` | 代码 linting |
| `rustfmt` | 代码格式化 |
| `rustdoc` | 文档生成 |
| `miri` | 未定义行为检测 |

## 在 OpenHarmony 中的定位

### 战略意义

Rust 工具链在 OpenHarmony 生态中扮演着**基础设施**角色：

```
┌─────────────────────────────────────────────────────────┐
│              OpenHarmony 系统构建流程                    │
├─────────────────────────────────────────────────────────┤
│                                                          │
│   开发者 → Cargo/Rustc → 目标二进制 → OH 镜像 → 设备     │
│                  ↑                                       │
│           Rust 工具链提供                                │
│                                                          │
└─────────────────────────────────────────────────────────┘
```

### 使用场景

1. **系统组件开发** - 使用 Rust 开发高性能系统组件
2. **应用开发** - 为 OHOS 设备开发 Rust 应用程序
3. **跨平台开发** - 利用 Rust 的跨平台特性统一多设备开发
4. **性能敏感模块** - 替代 C/C++ 开发安全关键模块

### 目标平台

| 目标三元组 | 适用设备 | 状态 |
|-----------|---------|------|
| `aarch64-unknown-linux-ohos` | 手机、平板、高端 IoT | ✅ 支持 |
| `armv7-unknown-linux-ohos` | 智能穿戴、低端 IoT | ✅ 支持 |
| `x86_64-unknown-linux-ohos` | x86 模拟器、测试设备 | ✅ 支持 |

## 与上游的差异

### 继承自上游

- Rust 编译器核心代码
- 标准库实现
- Cargo 包管理器
- 大部分工具链组件
- 测试套件

### OpenHarmony 特有修改

| 修改项 | 说明 |
|-------|------|
| 目标三元组 | 新增三个 `*-unknown-linux-ohos` 目标 |
| TLS 仿真 | 无原生 TLS，使用仿真方案 |
| 构建配置 | OHOS SDK 集成、Clang 包装器 |
| CI/CD | OHOS 特定构建和测试流程 |

## 相关资源

- **上游文档**: https://doc.rust-lang.org/
- **Cargo 手册**: https://doc.rust-lang.org/cargo/
- **Rust 书**: https://doc.rust-lang.org/book/
- **OpenHarmony 文档**: https://gitee.com/openharmony/docs
