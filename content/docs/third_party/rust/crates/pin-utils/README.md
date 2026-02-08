# pin-utils Wiki

**pin-utils** - OpenHarmony 第三方库集成文档

---

## 库概览

| 属性 | 内容 |
|-----|-----|
| **库名称** | pin-utils |
| **版本** | 0.1.0 |
| **上游地址** | https://github.com/rust-lang/pin-utils |
| **许可证** | Apache-2.0 OR MIT |
| **OH 组件名** | rust_pin_utils |
| **所属子系统** | thirdparty |

**一句话描述**: Rust Pin 类型工具宏库，提供 `pin_mut!`、`unsafe_pinned!` 等宏简化 pinned value 操作。

---

## OH 适配概述

### 适配状态: **零 Patch 集成**

```
┌─────────────────────────────────────────────────┐
│  适配复杂度: 极低                                 │
│  Patch 数量: 0                                   │
│  OH 特有代码: 无                                 │
│  特殊构建配置: 无                                │
└─────────────────────────────────────────────────┘
```

**关键特点**:
1. **无修改**: pin-utils 在 OH 中以原生形式使用，无任何 Patch
2. **标准构建**: 使用 `ohos_cargo_crate` 标准模板，无特殊配置
3. **单一依赖**: 仅被 nix 库的 AIO 功能依赖
4. **稳定可靠**: 官方 Rust 团队维护，无已知 CVE

### 文档导航

| 文档 | 内容 |
|-----|-----|
| [01_Overview.md](./01_Overview.md) | 库功能简介、OH 中的作用和定位 |
| [02_Patches.md](./02_Patches.md) | Patch 分析（本库无 Patch） |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 配置分析 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | OH 依赖关系和使用场景 |

### 快速定位

- **了解库功能** → [01_Overview.md](./01_Overview.md)
- **查看 OH 如何使用** → [04_Usage_in_OH.md](./04_Usage_in_OH.md)
- **构建配置详情** → [03_Build_Integration.md](./03_Build_Integration.md)

---

## 维护建议

| 建议项 | 说明 |
|-------|-----|
| **升级策略** | 可直接跟随上游升级，无 Patch 冲突风险 |
| **维护频率** | 低 - 代码稳定，无安全漏洞 |
| **监控重点** | nix 库的 AIO 功能是否继续依赖此库 |
| **潜在优化** | 如 nix 移除依赖，可考虑从 OH 移除 |

---

## 相关链接

- [上游仓库](https://github.com/rust-lang/pin-utils)
- [crates.io 页面](https://crates.io/crates/pin-utils)
- [API 文档](https://docs.rs/pin-utils)
- [Rust Pin 文档](https://doc.rust-lang.org/std/pin/index.html)

---

*本文档由 OpenHarmony Wiki Agent 自动生成*
*最后更新: 2026-02-08*
