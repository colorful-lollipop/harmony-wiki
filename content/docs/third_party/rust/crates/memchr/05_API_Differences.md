# API 与接口差异

## 概述

本节记录 memchr 库在 OpenHarmony 集成过程中与上游版本的 API 差异。由于该库采用上游原生版本集成，未进行任何 Patch 修改，因此 **API 与上游版本完全一致**。

## API 差异汇总

| API 项目 | 上游版本 | OpenHarmony 版本 | 差异说明 |
|----------|----------|------------------|----------|
| 所有公共 API | 2.5.0 | 2.5.0 | 无差异 |
| 公共结构体 | 全部可用 | 全部可用 | 无差异 |
| 公共 trait | 全部可用 | 全部可用 | 无差异 |
| Cargo 特性 | std（默认启用） | std（默认启用） | 无差异 |

## 详细对比

### 公共 API 一致性

memchr 库在 OpenHarmony 中提供的所有公共 API 与上游版本完全一致，包括：

#### 模块级 API

| API | 功能描述 | 状态 |
|-----|----------|------|
| `memchr::memchr` | 单字符正向搜索 | ✅ 一致 |
| `memchr::memrchr` | 单字符反向搜索 | ✅ 一致 |
| `memchr::memchr2` | 双字符搜索 | ✅ 一致 |
| `memchr::memchr3` | 三字符搜索 | ✅ 一致 |
| `memchr::memchr_iter` | 单字符搜索迭代器 | ✅ 一致 |
| `memchr::memchr2_iter` | 双字符搜索迭代器 | ✅ 一致 |
| `memchr::memchr3_iter` | 三字符搜索迭代器 | ✅ 一致 |
| `memmem::find` | 子字符串搜索 | ✅ 一致 |
| `memmem::rfind` | 子字符串反向搜索 | ✅ 一致 |
| `memmem::find_iter` | 子字符串搜索迭代器 | ✅ 一致 |
| `memmem::Finder` | 预构建搜索器 | ✅ 一致 |

#### 迭代器类型

| 类型 | 描述 | 状态 |
|------|------|------|
| `Memchr` | 单字符迭代器 | ✅ 一致 |
| `Memchr2` | 双字符迭代器 | ✅ 一致 |
| `Memchr3` | 三字符迭代器 | ✅ 一致 |

#### Cargo 特性

| 特性 | 默认启用 | 上游行为 | OH 行为 | 差异 |
|------|----------|----------|---------|------|
| `std` | 是 | 启用 CPU 特性检测 | 启用 CPU 特性检测 | 无 |
| `libc` | 否 | 可选使用 libc | 未启用 | 无 |
| `rustc-dep-of-std` | 否 | 内部使用 | 未使用 | 无 |

## OH 新增 API

本节记录 OpenHarmony 版本中新增的 API。

**结论**：该库未添加任何 OpenHarmony 特有的 API。所有 API 均来自上游版本。

## 行为变更 API

本节记录在上游基础上修改了行为的 API。

**结论**：该库不存在行为变更的 API。所有 API 在 OH 环境中的行为与上游完全一致。

## 废弃或禁用功能

本节记录在上游版本中存在但在 OpenHarmony 中被废弃或禁用的功能。

**结论**：该库未禁用任何上游功能。所有上游功能在 OH 环境中均可正常使用。

## 使用注意事项

### 跨平台兼容性

由于 API 完全一致，使用 memchr 库编写的代码在上游和 OpenHarmony 环境之间具有完全的移植性：

```rust
// 此代码在上游和 OH 环境中表现完全一致
use memchr::memchr;

let haystack = b"Hello, OpenHarmony!";
let pos = memchr(b'H', haystack);
assert_eq!(pos, Some(0));
```

### 特性一致性

在开发依赖 memchr 的 Rust 模块时，建议保持特性配置的一致性：

```gn
# 确保依赖链中特性配置一致
# 启用 std 特性以获得最佳性能
features = ["std"]
```

### 性能特性

在 OpenHarmony 中使用时，memchr 的性能特性与上游一致：

- **SIMD 加速**：在支持的 CPU 上自动启用
- **CPU 特性检测**：运行时自动检测并选择最优实现
- **平台自适应**：根据目标架构选择合适的代码路径

## 版本迁移说明

### 从上游迁移到 OH

如果您的代码已在上游环境中使用 memchr，迁移到 OpenHarmony 时：

1. **无需修改 API 调用**：所有 API 签名和行为完全一致
2. **无需修改依赖配置**：只需将依赖路径指向 OH 仓库
3. **无需修改特性配置**：特性行为保持一致

### 从 OH 迁移到上游

如果需要将 OH 中的代码迁移到上游环境：

1. **无需修改 API 调用**：所有 API 兼容
2. **只需更新 Cargo.toml 依赖**：将路径依赖改为版本号依赖
3. **无需修改代码逻辑**：行为完全一致

## 相关文档

- [01_Overview.md](./01_Overview.md) - 库功能概述
- [02_Patches.md](./02_Patches.md) - Patch 分析
- [03_Build_Integration.md](./03_Build_Integration.md) - 构建配置详解
- [04_Usage_in_OH.md](./04_Usage_in_OH.md) - 使用场景和依赖关系
