# proc-macro2

## 库概述

proc-macro2 是 Rust 生态中用于过程宏（Procedural Macro）开发的基础库，为 `proc_macro` 编译器 API 提供替代实现，使得基于 Token 的库可以脱离过程宏的特定使用场景运行。

在 OpenHarmony 中，该库作为 Rust 基础设施的核心依赖，被 quote、syn、cxx、bindgen 等关键 Rust crates 所使用。

## OpenHarmony 适配状态

| 项目 | 状态 |
|------|------|
| **Patch 数量** | 0 |
| **BUILD.gn 适配** | ✅ 完整适配 |
| **OH 特有修改** | 无 |
| **依赖者数量** | 8+ crates |

## 文档导航

- [01_Overview.md](01_Overview.md) - 库功能概述
- [02_Patches.md](02_Patches.md) - Patch 分析（本库无 Patch）
- [03_Build_Integration.md](03_Build_Integration.md) - 构建适配说明
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - 在 OH 中的使用方式

## 快速信息

- **上游版本**: 1.0.92
- **OH 版本**: 5.0
- **许可证**: MIT OR Apache-2.0
- **上游地址**: https://github.com/dtolnay/proc-macro2
- **最低 Rust 版本**: 1.56
