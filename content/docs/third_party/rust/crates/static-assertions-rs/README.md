# static-assertions-rs

## 库概览

static-assertions-rs 是一个 Rust 编译时断言宏库，提供在编译期验证常量、类型和 trait 实现的能力。

| 属性 | 值 |
|------|-----|
| **原始库名** | static_assertions |
| **版本** | 1.1.0 |
| **上游地址** | https://github.com/nvzqz/static-assertions-rs |
| **许可证** | MIT OR Apache-2.0 |
| **在 OH 中的角色** | 基础工具库（开发依赖） |

---

## OpenHarmony 适配概述

### 适配状态

| 适配项 | 状态 | 说明 |
|--------|------|------|
| Patch 数量 | ✅ 无 | 零 Patch 集成 |
| 特殊配置 | ✅ 无 | 标准模板构建 |
| OH 特有代码 | ✅ 无 | 纯上游代码 |
| API 变更 | ✅ 无 | 完全兼容上游 |

### 为什么需要这个库

在 OpenHarmony 的 Rust 生态中，static-assertions-rs 主要用于：

1. **编译时类型检查**: 确保类型实现特定 trait（如 `Send`、`Sync`）
2. **常量验证**: 验证编译期常量表达式的正确性
3. **API 兼容性测试**: 在开发阶段捕获类型不匹配问题

### 关键特性

该库提供的核心宏包括：

- **类型断言**: `assert_impl_all!`, `assert_impl_any!`, `assert_type_eq_all!`
- **常量断言**: `const_assert!`, `const_assert_eq!`, `const_assert_ne!`
- **大小断言**: `assert_eq_size!`, `assert_eq_align!`
- **Trait 断言**: `assert_obj_safe!`, `assert_trait_sub_all!`

---

## 文档导航

| 文档 | 内容 |
|------|------|
| [01_Overview.md](01_Overview.md) | 原始库简介与 OH 定位 |
| [02_Patches.md](02_Patches.md) | Patch 分析（本库无 Patch） |
| [03_Build_Integration.md](03_Build_Integration.md) | BUILD.gn 构建适配 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 在 OH 中的依赖关系与使用 |
| [05_API_Differences.md](05_API_Differences.md) | API 差异（无差异） |
| [06_Security.md](06_Security.md) | 安全风险分析 |

---

## 快速参考

### 在 Rust 中使用

```rust
// 确保类型实现 Send 和 Sync
assert_impl_all!(MyType: Send, Sync);

// 编译期常量断言
const_assert!(MAX_SIZE > 0);

// 类型大小相等断言
assert_eq_size!(TypeA, TypeB);
```

### 在 GN 中依赖

```gn
# 在 BUILD.gn 中添加依赖
deps = [
  "//third_party/rust/crates/static-assertions-rs:lib",
]
```

---

## 维护信息

| 项目 | 信息 |
|------|------|
| **评估日期** | 2026-02-08 |
| **评估版本** | 1.1.0 |
| **上游最新版本** | 1.1.0（稳定） |
| **维护建议** | 低风险，可直接升级 |

> **注意**: 本文档专注于 OpenHarmony 对该库的集成与适配，原始库功能请参考[上游文档](https://docs.rs/static_assertions/)。
