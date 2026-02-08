# memchr 库 OpenHarmony 集成文档

## 库概览

memchr 是一个提供高度优化的内存字符查找原语的 Rust 库。在 OpenHarmony 生态系统中，该库作为底层性能组件，为多个文本处理相关模块提供高效的字符搜索和子字符串匹配能力。

### 核心功能

memchr 库提供以下核心功能：

- **单字符快速搜索**：使用 SIMD 矢量指令加速单个字节的查找，支持正向（memchr）和反向（memrchr）搜索
- **多字符并行搜索**：支持同时搜索 2 个或 3 个不同的字节（memchr2、memchr3 及其变体）
- **子字符串搜索**：通过 `memmem` 模块提供高效的子字符串查找算法，包括 Two-Way 算法和基于 SIMD 的预过滤优化
- **跨平台优化**：在 x86_64 平台上自动检测 CPU 特性，支持 SSE2 和 AVX 加速；在其他平台上使用优化的通用实现

### OpenHarmony 定位

在 OpenHarmony 第三方库体系中，memchr 属于**基础性能层组件**。它不直接面向应用层，而是被其他 Rust crates（如 regex、aho-corasick、nom 等）依赖，为高级文本处理功能提供底层优化支持。这种定位使得 memchr 成为 OpenHarmony Rust 生态中性能敏感场景的关键基础设施。

## 文档导航

本 wiki 包含以下文档，帮助您全面了解 memchr 库在 OpenHarmony 中的集成和使用情况：

| 文档 | 描述 | 建议阅读对象 |
|------|------|-------------|
| [README.md](./README.md) | 本文档，提供库概览和文档导航 | 所有使用者 |
| [SUMMARY.md](./SUMMARY.md) | 阅读路线建议，根据角色推荐文档 | 新使用者 |
| [01_Overview.md](./01_Overview.md) | 原始库功能介绍和 OH 适配概述 | 需要了解基本功能的使用者 |
| [02_Patches.md](./02_Patches.md) | Patch 分析，说明无 Patch 的原因 | 维护者和升级相关人员 |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 构建配置详解 | 构建系统和集成开发者 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系和使用场景说明 | 想要了解如何使用的开发者 |

## 快速开始

### 在 OH 模块中使用 memchr

如果您的 Rust 模块需要依赖 memchr，请在 BUILD.gn 中添加以下依赖配置：

```gn
deps = ["//third_party/rust/crates/memchr:lib"]
```

### 基础使用示例

```rust
use memchr::memchr;

// 查找单个字符
let haystack = b"Hello, OpenHarmony!";
if let Some(pos) = memchr(b'H', haystack) {
    assert_eq!(pos, 0);
}

// 使用 memchr2 查找两个字符之一
use memchr::memchr2;
let positions: Vec<_> = memchr2_iter(b'a', b'b', b"abacabad").collect();
assert_eq!(positions, vec![0, 2, 4, 6]);
```

## 版本信息

| 项目 | 版本 |
|------|------|
| 上游版本 | 2.5.0 |
| OH 组件版本 | 6.1 |
| 上游地址 | https://github.com/BurntSushi/memchr |
| 许可证 | MIT / Unlicense |

## 相关资源

- [上游文档](https://docs.rs/memchr/)
- [上游仓库](https://github.com/BurntSushi/memchr)
- [ASSESSMENT.md](./_work/ASSESSMENT.md) - 详细的集成评估报告
