# rustc-hash

> OpenHarmony third_party 第三方库 Wiki 文档

## 库简介

**rustc-hash** 是 Rust 编译器（rustc）内部使用的高速非加密哈希算法库。在 OpenHarmony 中为 Rust 组件提供高性能的哈希能力。

| 项目 | 内容 |
|------|------|
| 库名称 | rustc-hash |
| 版本 | 1.1.0 |
| 许可证 | Apache-2.0 OR MIT |
| 上游地址 | https://github.com/rust-lang/rustc-hash |
| OH 组件 | rust_rustc_hash |
| 所属子系统 | thirdparty |

## 核心类型

- `FxHasher` - 快速非加密哈希算法实现
- `FxHashMap<K, V>` - 基于 FxHasher 的 HashMap（需 std 特性）
- `FxHashSet<V>` - 基于 FxHasher 的 HashSet（需 std 特性）

## OpenHarmony 适配概述

### Patch 状态

**✅ 无 Patch** - 该库完全使用上游代码，无需任何 OpenHarmony 定制即可运行。

### 构建适配

- 使用 `ohos_cargo_crate` 模板构建
- 配置与上游 Cargo.toml 高度一致
- 输出格式：`.rlib` 静态库

### 主要依赖者

| 模块 | 用途 |
|------|------|
| bindgen | Rust FFI 绑定生成工具，使用 FxHashMap 存储符号映射 |

## 文档导航

### 快速入门

1. [概述与 OH 定位](./01_Overview.md) - 了解该库在 OH 中的作用
2. [构建适配](./03_Build_Integration.md) - 了解构建配置详情

### 深入了解

- [OH 使用场景](./04_Usage_in_OH.md) - 了解依赖关系和使用方式
- [安全风险分析](./06_Security.md) - 了解安全使用建议

### 参考资料

- [阅读指南](./SUMMARY.md) - 推荐阅读路线
- [项目评估报告](./_work/ASSESSMENT.md) - 完整评估详情

## 使用示例

```rust
use rustc_hash::FxHashMap;
use rustc_hash::FxHashSet;

// 高性能哈希映射
let mut map: FxHashMap<u32, &str> = FxHashMap::default();
map.insert(1, "one");
map.insert(2, "two");

// 高性能哈希集合
let mut set: FxHashSet<&str> = FxHashSet::default();
set.insert("hello");
set.insert("world");
```

## 安全使用提醒

⚠️ **FxHashMap 不适用于处理不可信用户输入的场景**。对于需要 DOS 防护的场景，请使用标准库的 `HashMap`。

## 相关链接

- [上游 GitHub](https://github.com/rust-lang/rustc-hash)
- [crates.io](https://crates.io/crates/rustc-hash)
- [官方文档](https://docs.rs/rustc-hash/latest/rustc_hash/)
- [OH bundle.json](../bundle.json)
- [OH BUILD.gn](../BUILD.gn)
