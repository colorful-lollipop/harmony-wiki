# which-rs Wiki

## 库概览

**which-rs** 是 Unix `which` 命令的 Rust 实现，用于在系统 PATH 环境变量中查找可执行文件的路径。

| 属性 | 值 |
|------|-----|
| **上游版本** | 4.4.0 |
| **上游地址** | https://github.com/harryfei/which-rs |
| **许可证** | MIT |
| **OH 组件名** | rust_which_rs |
| **所属子系统** | thirdparty |

## OH 适配概述

### 关键特点

- ✅ **零 Patch**: 原生支持 OpenHarmony，无需任何修改
- ✅ **功能稳定**: 单一职责，API 简洁
- ✅ **维护简单**: 无定制代码，易于升级

### 与上游的主要差异

| 特性 | 上游 | OH |
|------|------|-----|
| regex feature | 可选启用 | **未启用** |
| Windows 支持 | 完整支持 | 未包含 |
| 开发依赖 | tempfile | 未引入 |

### 依赖关系

```
rust_which_rs
    ├── either (基础类型)
    └── libc (Unix 系统调用)
```

**OH 中使用情况**:
- bindgen 声明依赖 which-rs（用于查找 rustfmt）
- 实际代码中未使用，为遗留依赖

## 文档导航

### 快速开始

- [库概览](./01_Overview.md) - 了解 which-rs 的功能和 OH 定位
- [构建适配](./03_Build_Integration.md) - BUILD.gn 配置详解

### 深度分析

- [Patch 分析](./02_Patches.md) - Patch 分析（本库无 Patch）
- [依赖关系](./04_Usage_in_OH.md) - OH 中的依赖者和使用场景
- [API 差异](./05_API_Differences.md) - 可用 API 和特性差异
- [安全分析](./06_Security.md) - 安全风险评估和使用建议

### 开发文档

- [_work/ASSESSMENT.md](./_work/ASSESSMENT.md) - 项目评估报告
- [_work/NOTES.md](./_work/NOTES.md) - 分析过程记录
- [_work/PLAN.md](./_work/PLAN.md) - 任务规划

## 快速参考

### 基础用法

```rust
use which::which;

// 查找可执行文件
let path = which("rustc")?;
println!("Found at: {:?}", path);
```

### 在 OH 中使用

**GN 依赖**:
```gn
deps += [ "//third_party/rust/crates/which-rs:lib" ]
```

**Cargo.toml**:
```toml
[dependencies]
which = { path = "../../third_party/rust/crates/which-rs" }
```

## 维护信息

| 项目 | 内容 |
|------|------|
| **Patch 数量** | 0 |
| **维护复杂度** | 低 |
| **建议更新频率** | 跟随上游安全更新 |
| **已知 CVE** | 无 |

---

**文档版本**: 2026-02-07  
**维护者**: OpenHarmony Wiki Agent
