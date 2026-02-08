# either - Rust Either 类型库

> OpenHarmony 第三方库 Wiki  
> 路径: `third_party/rust/crates/either`

---

## 快速概览

| 属性 | 值 |
|------|-----|
| **上游名称** | either |
| **上游版本** | 1.8.1 |
| **上游地址** | https://github.com/bluss/either |
| **许可证** | Apache-2.0 OR MIT |
| **OH 组件名** | rust_either |
| **OH 组件版本** | 6.1 |
| **Patch 数量** | **0** |

### 一句话描述

**either** 是一个 Rust 基础库，提供 `Either<L, R>` 枚举类型，用于表示两种可能类型之一的值，类似于 `Result` 但不区分成功/失败语义。

---

## OpenHarmony 适配概述

### 适配方式

该库采用**零修改**方式集成到 OpenHarmony：

- ✅ **无 Patch**: 原始代码 100% 保留，无任何修改
- ✅ **标准 BUILD.gn**: 使用 `ohos_cargo_crate` 模板构建
- ✅ **无平台适配**: 纯 Rust 代码，跨平台兼容

### 构建配置

```gn
ohos_cargo_crate("lib") {
    crate_name = "either"
    crate_type = "rlib"
    features = ["use_std"]    # 启用标准库支持
    edition = "2018"
    cargo_pkg_version = "1.8.1"
}
```

### 关键差异

| 项目 | 上游 (Cargo) | OpenHarmony |
|------|-------------|-------------|
| 构建系统 | Cargo | GN + Ninja |
| 特性配置 | `features = ["use_std"]` | `features = ["use_std"]` |
| 可选依赖 | serde (未启用) | 未引入 serde |

---

## 文档导航

### 核心文档

| 文档 | 内容 |
|------|------|
| [01_Overview.md](./01_Overview.md) | 原始库功能介绍、在 OH 中的定位 |
| [02_Patches.md](./02_Patches.md) | Patch 分析 (本库无 Patch，说明原因) |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 详细说明、构建配置 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系、使用场景、依赖图 |
| [05_API_Differences.md](./05_API_Differences.md) | API 差异 (无差异) |
| [06_Security.md](./06_Security.md) | 安全分析、CVE 状态 |

### 工作文档

| 文档 | 内容 |
|------|------|
| [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) | Phase 0 信息收集报告 |
| [_work/NOTES.md](./_work/NOTES.md) | 分析过程笔记 |
| [_work/PLAN.md](./_work/PLAN.md) | 任务进度 |

---

## 在 OpenHarmony 中的角色

### 主要用途

1. **which-rs 依赖**: Unix `which` 命令的 Rust 实现依赖 either
2. **Rust 工具链**: Rust 编译器和 rust-analyzer 内部使用

### 典型使用场景

```rust
use either::Either;

// 表示两种可能的输入类型
fn process_input(input: Either<String, &str>) -> usize {
    either::for_both!(input, s => s.len())
}

// 使用示例
let owned = Either::Left("hello".to_string());
let borrowed = Either::Right("world");

assert_eq!(process_input(owned), 5);
assert_eq!(process_input(borrowed), 5);
```

---

## 维护说明

### 升级建议

- **可直接升级**: 无 patch，可直接替换为上游新版本
- **API 稳定**: 1.x 版本保持向后兼容
- **建议版本**: 与 Rust 工具链使用的 either 版本保持一致

### 联系人

- **上游维护**: https://github.com/bluss/either
- **OH 集成负责人**: fangting12@huawei.com

---

## 相关链接

- [上游仓库](https://github.com/bluss/either)
- [crates.io](https://crates.io/crates/either)
- [API 文档](https://docs.rs/either/1/)
