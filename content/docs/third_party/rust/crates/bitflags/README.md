# bitflags

> OpenHarmony Rust 位标志宏库集成文档

## 库概览

**bitflags** 是 Rust 生态系统中最流行的位标志（bitflags）宏库，用于简化位掩码操作。该库通过 `bitflags!` 宏生成具有良好语义和人体工程学 API 的标志枚举类型，广泛应用于系统编程、安全库、开发工具等多个领域。

在 OpenHarmony 生态中，bitflags 作为多个关键 Rust 库的基础依赖，承担着**基础设施**的角色，为 rustix、rust-openssl、clap、bindgen、nix 等核心库提供位标志支持。

## OpenHarmony 适配状态

| 项目 | 状态 | 说明 |
|------|------|------|
| **Patch 数量** | 0 | 无任何 OH 特有 Patch |
| **构建适配** | 部分平台 | 仅非 Linux ARM64 平台构建 |
| **依赖链** | 5 个直接依赖者 | 详见使用文档 |
| **原生兼容性** | 良好 | 无 std 依赖，支持跨平台 |

## 文档导航

### 核心文档

- **[01_Overview](01_Overview.md)** — 原始库功能介绍、版本信息、核心能力
- **[02_Patches](02_Patches.md)** — **Patch 分析记录**（本库无 Patch）
- **[03_Build_Integration](03_Build_Integration.md)** — OpenHarmony 构建系统适配详解
- **[04_Usage_in_OH](04_Usage_in_OH.md)** — 依赖关系、使用场景、集成方式
- **[05_API_Differences](05_API_Differences.md)** — API 差异分析（如有）
- **[06_Security](06_Security.md)** — 安全风险分析

### 工作文档

- **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** — 项目评估报告
- **[_work/NOTES.md](_work/NOTES.md)** — 分析过程记录
- **[_work/PLAN.md](_work/PLAN.md)** — 任务进度跟踪

## 快速参考

### 基本信息

```yaml
库名称: bitflags
版本: 2.9.1
许可证: MIT OR Apache-2.0
上游地址: https://github.com/bitflags/bitflags
OH 组件名: @ohos/rust_bitflags
所属子系统: thirdparty
```

### 依赖关系

```
应用层
    │
    ├── clap（命令行解析）
    │       └── bitflags ⬆️
    │
    ├── bindgen（绑定生成）
    │       └── bitflags ⬆️
    │
系统层 ────┤
    │
    ├── rustix（系统调用）
    │       └── bitflags ⬆️
    │
    ├── nix（Unix API）
    │       └── bitflags ⬆️
    │
    └── rust-openssl（加密库）
            └── bitflags ⬆️
```

### 关键适配点

1. **条件编译**：`host_os != "linux" || host_cpu != "arm64"` — Linux ARM64 平台不构建
2. **无 Patch**：该库无需任何修改即可在 OH 中使用
3. **crate_type**：rlib（静态库）

## 使用建议

### 适合阅读人群

- **Rust 开发者**：了解 bitflags 在 OH 中的集成方式
- **系统集成工程师**：分析依赖关系和构建配置
- **安全工程师**：评估位标志库的安全影响
- **版本升级负责人**：制定升级策略

### 阅读路线

| 角色 | 推荐阅读顺序 |
|------|--------------|
| 快速了解 | README → 01_Overview → 03_Build_Integration |
| 深度集成 | README → 全部文档按顺序阅读 |
| 安全评估 | README → 06_Security → 04_Usage_in_OH |
| 升级准备 | README → 02_Patches → 03_Build_Integration → 06_Security |

## 版本信息

| 版本 | 日期 | 变更 |
|------|------|------|
| 2.9.1 | - | 当前 OH 集成版本 |
| 2.6.0 | - | 上游早期版本（参考） |

## 相关资源

- **上游文档**：[bitflags - docs.rs](https://docs.rs/bitflags)
- **上游仓库**：[bitflags/bitflags](https://github.com/bitflags/bitflags)
- **规范文档**：[spec.md](./spec.md)（上游）
- **OH 第三方库索引**：[third_party/rust/crates](../README.md)

## 贡献指南

本 Wiki 由 OpenHarmony 第三方库维护团队生成。如发现文档错误或需要更新，请联系组件 Owner 或提交 Issue。

---

*最后更新：2024年*
*文档版本：1.0*
