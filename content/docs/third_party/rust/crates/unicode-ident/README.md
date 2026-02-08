# unicode-ident Wiki

**unicode-ident** 在 OpenHarmony 中的集成与适配文档。

---

## 快速导航

| 文档 | 内容 |
|------|------|
| [01_Overview.md](01_Overview.md) | 库概览、功能介绍、OH 适配概述 |
| [02_Patches.md](02_Patches.md) | Patch 分析（本库无 Patch） |
| [03_Build_Integration.md](03_Build_Integration.md) | BUILD.gn 配置详解 |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | OH 中的依赖关系与使用场景 |

---

## 库概览

### 基本信息

| 属性 | 值 |
|------|-----|
| **名称** | unicode-ident |
| **版本** | 1.0.14 |
| **上游** | https://github.com/dtolnay/unicode-ident |
| **许可证** | (MIT OR Apache-2.0) AND Unicode-3.0 |
| **功能** | Unicode 标识符验证（XID_Start / XID_Continue） |

### OpenHarmony 适配状态

| 检查项 | 状态 |
|--------|------|
| **OH 特定 Patch** | ❌ 无 |
| **特殊适配代码** | ❌ 无 |
| **标准 BUILD.gn** | ✅ 是 |
| **依赖复杂度** | 叶子节点（无依赖） |

---

## 核心发现

### 1. 无 Patch 设计

unicode-ident **没有任何 OpenHarmony Patch**，这是**设计使然**：

- 纯数据表实现（`src/tables.rs`）
- 无平台相关代码
- 使用 `#![no_std]`
- 直接实现 Unicode 标准

### 2. 基础依赖地位

作为 Rust 过程宏生态的基石：

```
unicode-ident
    └── proc-macro2
        └── syn
            ├── serde_derive (序列化)
            ├── clap_derive (CLI)
            ├── bindgen (FFI)
            └── ... (更多 derive 宏)
```

**影响范围**: 约 90% 的 Rust 代码间接依赖此库。

### 3. 极简 BUILD.gn

```gn
ohos_cargo_crate("lib") {
    crate_name = "unicode_ident"
    crate_type = "rlib"
    crate_root = "src/lib.rs"
    sources = ["src/lib.rs"]
    edition = "2018"
    # 无 deps、无 features、无 build.rs
}
```

这是 OpenHarmony Rust crate 的**最简标准模板**。

---

## 文档导航建议

### 初次了解
1. 阅读本页（README.md）
2. 查看 [01_Overview.md](01_Overview.md) 了解库功能

### 深入了解 OH 集成
1. [03_Build_Integration.md](03_Build_Integration.md) - BUILD.gn 详解
2. [04_Usage_in_OH.md](04_Usage_in_OH.md) - 依赖关系分析

### Patch 维护者
- [02_Patches.md](02_Patches.md) - 说明为何无 Patch

---

## 维护信息

| 项目 | 信息 |
|------|------|
| **评估日期** | 2025-02-07 |
| **文档版本** | 1.0 |
| **上游版本** | 1.0.14 |
| **Unicode 版本** | 16.0.0 |

---

## 相关链接

- **原始库**: https://github.com/dtolnay/unicode-ident
- **文档**: https://docs.rs/unicode-ident/1.0.14/
- **Unicode TR31**: https://www.unicode.org/reports/tr31/

---

*本文档遵循 OpenHarmony 第三方库 Wiki 规范生成。*
