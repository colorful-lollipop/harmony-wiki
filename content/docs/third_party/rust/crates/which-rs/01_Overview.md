# which-rs 概述

## 基本信息

| 属性 | 值 |
|------|-----|
| **库名称** | which |
| **上游版本** | 4.4.0 |
| **上游仓库** | https://github.com/harryfei/which-rs |
| **许可证** | MIT |
| **crates.io** | https://crates.io/crates/which |

## 功能简介

which-rs 是 Unix `which` 命令的 Rust 实现，用于在系统 PATH 环境变量中查找可执行文件的路径。

### 核心功能

1. **查找可执行文件**: 给定程序名，返回其在 PATH 中的完整路径
2. **支持多种查找模式**:
   - 绝对路径验证
   - 相对路径解析
   - PATH 环境变量遍历
3. **跨平台支持**: Linux、macOS、Windows
4. **可选正则匹配**: 支持使用正则表达式批量查找（regex feature）

### 代码示例

```rust
use which::which;
use std::path::PathBuf;

// 查找 rustc 可执行文件
let result = which("rustc").unwrap();
assert_eq!(result, PathBuf::from("/usr/bin/rustc"));
```

## 在 OpenHarmony 中的定位

### 组件信息

| 属性 | 值 |
|------|-----|
| **OH 组件名** | rust_which_rs |
| **OH 版本** | 6.1 |
| **所属子系统** | thirdparty |
| **组件负责人** | fangting12@huawei.com |

### 在 OH 中的作用

which-rs 在 OpenHarmony 中作为**基础工具库**使用，主要用于：

1. **构建工具链**: 在编译时查找各种工具（如 bindgen 查找 clang）
2. **运行时工具定位**: 帮助应用程序找到系统命令的位置
3. **跨平台兼容性**: 提供统一的接口在不同平台上查找可执行文件

### 为何选择该库

- **轻量级**: 无复杂依赖，核心功能仅需 `either` 和 `libc`
- **稳定**: API 设计简洁，版本迭代平缓
- **跨平台**: 原生支持 OH 标准系统基于的 Linux 平台
- **许可证友好**: MIT 许可证无商用限制

## 源代码结构

```
src/
├── lib.rs      # 公共 API：which(), which_in(), Path, CanonicalPath, WhichConfig
├── finder.rs   # 查找实现：PATH 遍历、扩展名处理
├── checker.rs  # 可执行检查：Unix 权限检查、文件存在性检查
├── error.rs    # 错误类型：CannotFindBinaryPath 等
└── helper.rs   # Windows 辅助函数（条件编译）
```

### 关键模块说明

#### lib.rs - 公共接口
- `which(binary_name)`: 查找单个可执行文件
- `which_all(binary_name)`: 查找所有匹配项
- `which_in(binary_name, paths, cwd)`: 在指定路径中查找
- `Path`: 包装类型，表示已验证的可执行文件路径
- `CanonicalPath`: 包装类型，表示规范化的可执行文件路径
- `WhichConfig`: 配置化查找构建器

#### finder.rs - 查找逻辑
- 处理绝对路径、相对路径、PATH 查找三种场景
- Windows 平台处理 PATHEXT 环境变量
- 可选的正则表达式匹配功能

#### checker.rs - 验证逻辑
- `ExecutableChecker`: Unix 使用 `libc::access(path, X_OK)` 检查可执行权限
- `ExistedChecker`: 检查文件存在性
- `CompositeChecker`: 组合多个检查器

## 与上游的差异

### OH 构建配置

| 特性 | 上游 Cargo.toml | OH BUILD.gn | 说明 |
|------|-----------------|-------------|------|
| regex feature | 可选启用 | **未启用** | 减少依赖 |
| Windows 支持 | 完整支持 | 未包含 | OH 基于 Linux |
| 测试依赖 | tempfile | 未引入 | 仅构建库 |

### 版本状态

- **当前集成版本**: 4.4.0
- **上游最新版本**: 6.x（建议关注升级）

## 相关文档

- [Patch 分析](./02_Patches.md) - 详细的 Patch 分析（本库无 Patch）
- [构建适配](./03_Build_Integration.md) - BUILD.gn 详解
- [OH 中的使用](./04_Usage_in_OH.md) - 依赖关系和使用场景
- [安全分析](./06_Security.md) - 安全风险评估
