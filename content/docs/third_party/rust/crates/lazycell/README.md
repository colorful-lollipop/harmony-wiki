# lazycell OpenHarmony 集成文档

> **lazycell**: 提供延迟初始化 Cell 的 Rust 库
>
> **OH 组件**: @ohos/rust_lazycell v6.1
>
> **上游版本**: 1.2.1 (1.3.0 in BUILD.gn)
>
> **集成状态**: ✅ 零修改适配（最佳实践）

---

## 文档说明

本文档说明 lazycell Rust crate 在 OpenHarmony 中的集成与适配情况。

**重点内容**:
- OH 中的零 Patch 集成最佳实践
- BUILD.gn 构建适配
- 依赖关系（bindgen、compiletest）
- 迁移到现代替代方案的路径

**无需关注**:
- lazycell 的原始功能（简要说明即可，详见上游文档）

---

## 快速导航

### 入门阅读

1. **[SUMMARY.md](SUMMARY.md)** — 推荐阅读路线
2. **[01_Overview.md](01_Overview.md)** — 原始库简介
3. **[04_Usage_in_OH.md](04_Usage_in_OH.md)** — OH 中的使用情况

### OH 集成详解

1. **[02_Patches.md](02_Patches.md)** — Patch 分析（无 Patch）
2. **[03_Build_Integration.md](03_Build_Integration.md)** — OH 构建适配

### 进阶内容

1. **[05_Migration_Guide.md](05_Migration_Guide.md)** — 迁移到 once_cell / LazyLock

### 工作记录

- **[_work/ASSESSMENT.md](_work/ASSESSMENT.md)** — 完整的项目评估报告

---

## 关键发现

| 维度 | 结论 |
|------|------|
| **集成复杂度** | 极低（无 Patch、无修改） |
| **OH 依赖者** | 2 个（bindgen、compiletest） |
| **维护成本** | 低（无本地修改） |
| **推荐状态** | ✅ 现有代码可继续使用<br/>⚠️ 新代码建议替代方案 |

---

## 一句话总结

**lazycell 是 OH 第三方库集成的最佳实践案例：零 Patch、零修改、零特殊配置，仅通过 BUILD.gn 完成构建系统桥接。**

---

## OH 适配概述

### 零 Patch 集成

lazycell 是 OH 中最干净的第三方库集成之一：

- ✅ **无源码修改**: 650 行源码完全与上游一致
- ✅ **无 Patch 文件**: 无任何 `.patch` 文件
- ✅ **无特殊配置**: BUILD.gn 仅使用 `ohos_cargo_crate` 模板
- ✅ **无条件编译**: 无 `#[cfg(ohos)]` 或 OH 特定宏

### BUILD.gn 适配

```gn
ohos_cargo_crate("lib") {
    crate_name = "lazycell"
    crate_type = "rlib"
    crate_root = "src/lib.rs"
    sources = ["src/lib.rs"]
    edition = "2015"
    cargo_pkg_version = "1.3.0"
    part_name = "rust_lazycell"
    subsystem_name = "thirdparty"
}
```

**关键点**:
- 使用 `ohos_cargo_crate` 适配 Rust 构建
- 输出 `rlib`（Rust 静态库），仅供 Rust 代码使用
- 无额外依赖、编译选项或特性开关

---

## OH 中的使用

### 依赖者

| 模块 | 用途 |
|------|------|
| **bindgen** | 自动生成 Rust FFI 绑定到 C/C++ 库 |
| **compiletest** | Rust 编译器测试工具，使用 AtomicLazyCell 延迟初始化目标配置 |

### 典型使用场景

```rust
// compiletest 中的典型用法
static target_cfgs: AtomicLazyCell<TargetCfgs> = AtomicLazyCell::NONE;

fn get_target_cfgs() -> &'static TargetCfgs {
    target_cfgs.borrow_with(|| {
        // 计算昂贵的配置
        compute_target_cfgs()
    })
}
```

**场景**: 延迟初始化目标平台配置，避免重复计算，支持线程安全访问。

---

## 替代方案

| 方案 | 稳定性 | 推荐度 |
|------|--------|--------|
| **lazycell** | 稳定 | ★★★★☆（现有代码） |
| **once_cell** | 稳定 | ★★★★★（新代码，功能更丰富） |
| **std::sync::LazyLock** | Rust 1.80+ | ★★★★★（标准库，无外部依赖） |

**建议**: OH 内新代码优先使用 `once_cell` 或标准库方案。

---

## 文档结构

```
wiki/
├── README.md                    # 本文件
├── SUMMARY.md                   # 阅读路线建议
├── _work/
│   ├── ASSESSMENT.md            # 完整评估报告
│   ├── NOTES.md                 # 分析过程记录
│   └── PLAN.md                  # 任务进度
├── 01_Overview.md               # 原始库简介
├── 02_Patches.md                # Patch 分析
├── 03_Build_Integration.md      # OH 构建适配
├── 04_Usage_in_OH.md            # 依赖关系与使用
└── 05_Migration_Guide.md        # 迁移指南
```

---

## 贡献指南

本文档由 OpenHarmony Wiki Agent 自动生成。

如有疑问或建议，请联系组件所有者：`fangting12@huawei.com`

---

## 参考资料

- **上游仓库**: https://github.com/indiv0/lazycell
- **API 文档**: https://indiv0.github.io/lazycell/lazycell
- **Cargo 页面**: https://crates.io/crates/lazycell
- **OH 组件**: @ohos/rust_lazycell

---

## 许可证

本文档遵循 Apache License 2.0。

lazycell 原始库采用 MIT / Apache-2.0 双重许可。
