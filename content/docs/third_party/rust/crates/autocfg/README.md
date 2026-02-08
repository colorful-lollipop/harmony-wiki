# autocfg - OpenHarmony Wiki

> **版本**: 1.4.0  
> **上游**: https://github.com/cuviper/autocfg  
> **OH 组件**: @ohos/rust_autocfg

---

## 快速概览

**autocfg** 是 Rust 生态的基础工具库，用于在**构建时**自动探测 Rust 编译器支持的特性。它在 OpenHarmony 中作为 **零 Patch 第三方库** 引入，仅需标准的 BUILD.gn 构建适配，无需任何代码修改。

### 核心特点

| 特性 | 说明 |
|-----|------|
| **零 Patch** | 完全使用上游代码，无 OH 特定修改 |
| **构建时工具** | 不进入运行时，只在编译时使用 |
| **平台无关** | 基于 Rust 编译器接口，与操作系统无关 |
| **稳定成熟** | 1.0+ 版本，API 稳定，兼容 Rust 1.0+ |

### 在 OpenHarmony 中的作用

```
Rust 应用/库 → build.rs → autocfg → 探测 rustc 特性 → 条件编译代码
```

autocfg 被其他 Rust crate（如 memoffset）在 `build.rs` 构建脚本中使用，用于：
- 探测编译器版本（如是否支持 Rust 1.50+）
- 探测特定类型/特性/路径是否存在
- 生成 `cargo:rustc-cfg=XXX` 配置标志

---

## 文档导航

### 必读文档

| 文档 | 内容 | 推荐顺序 |
|------|------|---------|
| [01_Overview.md](./01_Overview.md) | 原始库简介、功能说明、在 OH 中的定位 | 1 |
| [02_Patches.md](./02_Patches.md) | Patch 分析（本库无 Patch，说明原因） | 2 |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 构建适配详解 | 3 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系、使用场景、依赖图 | 4 |

### 选读文档

| 文档 | 内容 |
|------|------|
| [05_API_Differences.md](./05_API_Differences.md) | API/接口差异（本库无差异） |
| [06_Security.md](./06_Security.md) | 安全风险分析 |

### 工作文档

| 文档 | 说明 |
|------|------|
| [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) | 项目评估报告（信息收集结果） |
| [_work/NOTES.md](./_work/NOTES.md) | 分析过程记录 |
| [_work/PLAN.md](./_work/PLAN.md) | 任务进度计划 |

---

## 关键信息速查

### 基础信息

```yaml
库名称: autocfg
上游版本: 1.4.0 (2024-09-26)
上游地址: https://github.com/cuviper/autocfg
许可证: Apache-2.0 OR MIT

OH 组件名: rust_autocfg
OH 包名: @ohos/rust_autocfg
子系统: thirdparty
维护人: fangting12@huawei.com
```

### Patch 状态

| 项目 | 状态 |
|------|------|
| Patch 数量 | **0** |
| OH 特定代码 | **无** |
| 修改类型 | **无需修改** |

### 依赖关系

**下游依赖者**（在 OH 中）：
- [memoffset](../memoffset) - 探测 `offset_of!` 宏支持

**依赖类型**：均为 `build-dependencies`（构建时依赖）

---

## 为什么 autocfg 不需要 Patch？

1. **功能单一明确** - 仅探测编译器特性，无平台相关逻辑
2. **平台无关** - 基于标准 Rust 编译器接口
3. **构建时使用** - 不进入目标运行时，不依赖 OS 特性
4. **成熟稳定** - 1.0+ 版本，功能完备，无需扩展

---

## 维护注意事项

### 升级策略

- ✅ 可直接同步上游新版本
- ✅ 无需 Patch 迁移
- ✅ 遵循 semver，升级安全

### 注意事项

- autocfg 是**构建时依赖**，升级不会增加运行时体积
- 保持与上游版本同步即可，无需额外维护

---

## 相关链接

- **上游仓库**: https://github.com/cuviper/autocfg
- **Crates.io**: https://crates.io/crates/autocfg
- **API 文档**: https://docs.rs/autocfg
- **OH 构建模板**: `//build/templates/rust/ohos_cargo_crate.gni`

---

*本文档由 OpenHarmony Wiki Agent 自动生成于 2025-02-08*
