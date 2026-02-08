# rustc-hash 库概述

## 基础信息

| 项目 | 内容 |
|------|------|
| 库名称 | rustc-hash |
| 版本 | 1.1.0 |
| 许可证 | Apache-2.0 OR MIT（双许可证） |
| 上游地址 | https://github.com/rust-lang/rustc-hash |
| OH 组件名称 | rust_rustc_hash |
| 所属子系统 | thirdparty |
| 代码规模 | 149 行（核心实现） |

## 原始功能简介

rustc-hash 是 Rust 编译器（rustc）内部使用的高速非加密哈希算法库。该库实现了以下核心类型：

- **FxHasher**：基于 Firefox 和 rustc 内部哈希算法的 64 位非加密哈希实现
- **FxHashMap<K, V>**：使用 FxHasher 的 HashMap 类型别名
- **FxHashSet<V>**：使用 FxHasher 的 HashSet 类型别名

该算法的设计目标是在**性能优先、不考虑 DOS 攻击**的场景下提供最快的哈希速度。根据 rustc 团队的测试，该算法在编译器内部工作负载上表现优于 FNV 哈希——碰撞率相似或略差，但哈希函数本身的速度显著更高，因为它一次可以处理最多 8 个字节的数据。

## 该库在 OpenHarmony 中的定位

### 定位说明

rustc-hash 在 OpenHarmony 生态中定位为**基础性能优化组件**，为 Rust 开发者提供高速的非加密哈希能力。

### 作用与价值

1. **构建工具支持**：为 bindgen（Rust FFI 绑定生成工具）提供高性能哈希，加速 C/C++ 绑定的生成过程
2. **Rust 生态集成**：作为 Rust 官方推荐的快速哈希库，是 OH 接入更广泛 Rust 生态的基础组件
3. **性能优化基础**：为未来可能需要高性能哈希的 OH 组件提供基础设施

### 在 OH 中的特殊地位

与其他大型第三方库不同，rustc-hash 在 OH 中具有以下特点：

- **极简集成**：无需任何 Patch，完全使用上游代码
- **单一依赖**：目前仅被 bindgen 直接依赖
- **基础组件**：虽然依赖者不多，但它是 OH Rust 生态的重要组成部分

## 版本信息

### OH 版本信息

根据 bundle.json 配置：
- OH 版本标识：6.1
- 发布类型：code-segment
- 目标路径：third_party/rust/crates/rustc-hash

### 上游版本

上游 crates.io 版本：1.1.0，与 OH 当前版本一致。

### 版本建议

**注意**：该库使用 Rust 2015 edition，这是相对较旧的版本。建议在后续升级中考虑迁移至更新的 Rust edition（如 2018 或 2021）以获得更好的语言特性和更长的维护支持。

## Cargo 特性

| 特性 | 状态 | 说明 |
|------|------|------|
| std | 启用（默认） | 启用标准库支持，提供 FxHashMap 和 FxHashSet 类型别名 |
| default-features | 是 | 默认启用 std 特性 |

## 快速使用示例

```rust
use rustc_hash::FxHashMap;
use rustc_hash::FxHashSet;

// 创建 FxHashMap
let mut map: FxHashMap<u32, String> = FxHashMap::default();
map.insert(1, "one".to_string());
map.insert(2, "two".to_string());

// 创建 FxHashSet
let mut set: FxHashSet<&str> = FxHashSet::default();
set.insert("hello");
set.insert("world");

// 查找操作
if let Some(value) = map.get(&1) {
    println!("Found: {}", value);
}
```

## 与标准库 HashMap 的对比

| 特性 | FxHashMap (rustc-hash) | HashMap (std) |
|------|------------------------|---------------|
| 哈希算法 | FxHasher (非加密) | SipHash 1-3 (加密级) |
| DOS 防护 | 无 | 有 |
| 速度 | 非常快 | 较慢 |
| 适用场景 | 可信数据、编译器内部 | 不可信输入、网络数据 |
| 确定性 | 是 | 否（随机化） |

## 相关文档

- [构建适配详情](./03_Build_Integration.md)
- [OH 使用情况](./04_Usage_in_OH.md)
- [安全风险分析](./06_Security.md)
- [上游官方文档](https://docs.rs/rustc-hash/latest/rustc_hash/)
