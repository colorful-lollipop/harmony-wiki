# `peeking_take_while` - OpenHarmony 集成文档

> OpenHarmony 第三方库 `peeking_take_while` Rust Crate 文档
>
> **版本**: 0.1.2 (BUILD.gn) / 1.0.0 (Cargo.toml) ⚠️
> **许可证**: Apache-2.0 OR MIT
> **上游**: https://github.com/fitzgen/peeking_take_while

---

## 📖 概述

`peeking_take_while` 是一个提供**迭代器适配器**的 Rust 库，提供类似标准库 `take_while` 的功能，但关键区别在于：**通过预查看（peek）下一个元素来避免消耗首个不满足条件的元素**。

### 在 OpenHarmony 中的角色

- **类别**: 基础 Rust 工具库
- **子系统**: thirdparty
- **组件**: rust_peeking_take_while
- **适配状态**: ✅ 纯上游引入，无 Patch

### 核心价值

解决标准库 `Iterator::take_while` 的一个关键限制：
- `take_while` 会**消费**第一个使 predicate 返回 `false` 的元素
- `peeking_take_while` 通过**预查看（peek）**避免丢失该元素

---

## 🎯 快速导航

### 新手入门

1. **[SUMMARY.md](./SUMMARY.md)** - 推荐的阅读路线
2. **[01_Overview.md](./01_Overview.md)** - 原始库简介和功能说明
3. **[03_Build_Integration.md](./03_Build_Integration.md)** - OH 构建系统适配

### 深入了解

4. **[04_Usage_in_OH.md](./04_Usage_in_OH.md)** - 依赖关系和使用场景
5. **[06_Security.md](./06_Security.md)** - 安全风险分析

### 工作文档

- **[_work/ASSESSMENT.md](./_work/ASSESSMENT.md)** - 项目评估结果
- **[_work/PLAN.md](./_work/PLAN.md)** - 文档生成任务计划
- **[_work/NOTES.md](./_work/NOTES.md)** - 分析过程记录

---

## 🚀 快速开始

### 基本用法

```rust
use peeking_take_while::PeekableExt;

let mut iter = (0..100).peekable();

// 使用 peeking_take_while 处理小于 10 的元素
let sum: u32 = iter.by_ref()
    .peeking_take_while(|&x| x < 10)
    .sum();
assert_eq!(sum, 45);

// 元素 10 被保留！
assert_eq!(iter.next(), Some(10));

// 而标准 take_while 会消耗元素 10
let mut iter2 = (0..100).peekable();
let _sum2: u32 = iter2.by_ref().take_while(|&x| x < 10).sum();
assert_eq!(iter2.next(), Some(11));  // 10 被消耗掉了！
```

### 在 OpenHarmony 中使用

```toml
# Cargo.toml
[dependencies]
peeking_take_while = "0.1.2"  # 或 1.0.0（建议升级）
```

或在 BUILD.gn 中依赖：

```gn
deps = [
    "//third_party/rust/crates/peeking_take_while:lib",
]
```

---

## 📦 核心特性

| 特性 | 状态 | 说明 |
|------|------|------|
| **no_std 支持** | ✅ | 适合嵌入式和 OpenHarmony 场景 |
| **零 unsafe 代码** | ✅ | `forbid(unsafe_code)` |
| **零 Patch** | ✅ | 完全使用上游代码 |
| **完整文档** | ✅ | 包含详细注释和示例 |
| **测试覆盖** | ✅ | 5 个测试用例 |

---

## ⚠️ 已知问题

### 版本不一致

| 文件 | 版本 | Edition |
|------|------|---------|
| BUILD.gn | 0.1.2 | 2015 |
| Cargo.toml | 1.0.0 | 2018 |

**影响**: 可能导致版本混乱和用户困惑

**建议**: 统一为 1.0.0 版本

### 上游维护状态

- 最后一次提交: 2021-09-03
- 维护状态: 低维护模式

**风险**: 低（代码简单且稳定）

---

## 🔄 版本升级建议

### 当前状态
- OH 使用版本: 0.1.2 (2017-05-17)
- 上游最新版本: 1.0.0 (2021-09-03)

### 升级到 1.0.0 的优势

1. ✅ **Rust 2018 Edition** - 更好的语言特性支持
2. ✅ **no_std 支持** - 明确声明，适合 OH 环境
3. ✅ **改进的文档** - 更清晰的说明和示例
4. ✅ **优化实现** - 使用 `Iterator::next_if` 更简洁

### 升级影响

- **API 兼容**: ✅ 完全兼容
- **代码改动**: 仅配置文件，无需修改源码

---

## 📚 相关资源

### 官方资源
- **上游仓库**: https://github.com/fitzgen/peeking_take_while
- **crates.io**: https://crates.io/crates/peeking_take_while
- **文档**: https://docs.rs/peeking_take_while

### OpenHarmony 资源
- **组件**: @ohos/rust_peeking_take_while
- **路径**: `third_party/rust/crates/peeking_take_while`
- **许可证**: Apache-2.0 OR MIT

### 替代方案
- **itertools**: https://github.com/rust-itertools/itertools
  - 功能更全面
  - 维护更活跃
  - 但依赖更大

---

## 📝 文档说明

### 文档范围

本文档重点说明：
- ✅ OpenHarmony 对该库的适配方式
- ✅ 构建系统配置
- ✅ 依赖关系和使用场景
- ✅ 安全性和合规性

**不包含**：
- ❌ 上游原始文档的重复内容
- ❌ Rust 迭代器的通用教程
- ❌ 与 OpenHarmony 无关的内容

### 文档规范

- **语言**: 中文（保留英文术语）
- **证据优先**: 所有结论基于直接证据
- **标注**: 不确定处标注 `TODO(需确认)`

---

## 🤝 贡献

如发现文档错误或需要补充，请联系：
- **负责人**: fangting12@huawei.com
- **子系统**: thirdparty

---

**文档版本**: 1.0.0
**最后更新**: 2026-02-08
**维护者**: OpenHarmony Wiki Agent
