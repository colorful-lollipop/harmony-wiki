# strsim-rs

> OpenHarmony 第三方库 Wiki 文档

## 库概述

**strsim-rs** 是一个纯 Rust 实现的字符串相似度计算库，提供多种字符串距离和相似度算法。该库在 OpenHarmony 中作为基础工具库，通过 **clap** 命令行参数解析器间接为 Rust 命令行工具提供智能提示功能。

| 项目 | 值 |
|------|-----|
| **版本** | 0.10.0 (OH: 6.1) |
| **许可证** | Apache 2.0, MIT |
| **上游地址** | https://github.com/dguo/strsim-rs |
| **OH 组件** | @ohos/rust_strsim_rs |
| **分类** | 字符串处理 / 算法库 |

## 核心功能

strsim-rs 实现了以下字符串相似度算法：

- **Hamming 距离**：计算等长字符串对应位置字符不同的数量
- **Levenshtein 距离**：计算字符串转换所需的最少编辑次数（插入、删除、替换）
- **OSA 距离**：允许相邻字符换位的 Levenshtein 变体
- **Damerau-Levenshtein 距离**：支持完整换位的真正算法
- **Jaro / Jaro-Winkler 相似度**：模糊匹配算法，对共同前缀更敏感
- **Sørensen-Dice 相似度**：基于字符 bigram 的相似度计算

## OpenHarmony 适配状态

| 适配维度 | 状态 | 说明 |
|----------|------|------|
| **Patch 文件** | 无需 | 该库为纯算法库，无平台相关代码 |
| **构建适配** | 标准 | 使用 `ohos_cargo_crate` 模板 |
| **OH 特有依赖** | 无 | 仅依赖 Rust 标准库 |
| **源代码修改** | 无 | 源代码与上游完全一致 |

## 依赖关系

```
strsim-rs (提供字符串相似度算法)
    ↓
clap (命令行参数解析器)
    ↓
Rust 命令行工具 (间接使用)
```

**直接依赖者**：
- `clap` - 命令行参数解析库，用于 `suggestions` 特性

## 文档导航

| 文档 | 内容 | 阅读建议 |
|------|------|----------|
| [SUMMARY.md](./SUMMARY.md) | 文档结构和阅读路线 | 首次阅读请先浏览 |
| [01_Overview.md](./01_Overview.md) | 原始库功能介绍 | 了解库的基本功能 |
| [02_Patches.md](./02_Patches.md) | Patch 分析 | 重点关注 OH 适配差异 |
| [03_Build_Integration.md](./03_Build_Integration.md) | 构建配置说明 | 开发者必读 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | OH 使用情况 | 了解依赖关系和使用场景 |

## 快速使用示例

```rust
use strsim::{levenshtein, jaro_winkler, sorensen_dice};

// Levenshtein 距离
let dist = levenshtein("kitten", "sitting");  // 返回 3

// Jaro-Winkler 相似度 (0.0 - 1.0)
let similarity = jaro_winkler("hello", "hallo");  // 返回 ~0.84

// Sørensen-Dice 相似度
let dice = sorensen_dice("web applications", "applications of the web");  // 返回 ~0.79
```

## 版本信息

| 版本 | 日期 | 变更 |
|------|------|------|
| 0.10.0 | 2023 | 上游当前版本 |
| 6.1 (OH) | - | OH 打包版本号 |

## 相关资源

- **上游文档**：https://docs.rs/strsim/
- **上游源码**：https://github.com/dguo/strsim-rs
- **OH 组件**：bundle.json 中的 @ohos/rust_strsim_rs
- **构建配置**：BUILD.gn

## 维护建议

1. **上游升级**：可直接升级上游版本，无需 OH 特定修改
2. **安全更新**：该库不涉及安全敏感操作，可随 clap 升级周期更新
3. **测试**：上游测试用例可直接使用，无需 OH 特定测试

---

*文档最后更新：2026-02-08*
