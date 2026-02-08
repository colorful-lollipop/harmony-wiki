# glob - OpenHarmony 集成文档

[![License](https://img.shields.io/badge/license-Apache%202.0%20%7C%20MIT-blue)](LICENSE-APACHE)
[![Version](https://img.shields.io/badge/version-0.3.1-orange)](https://github.com/rust-lang/glob)
[![OH Version](https://img.shields.io/badge/OH-6.1-green)](bundle.json)

## 库概览

**glob** 是一个提供 Unix shell 风格 glob 模式匹配支持的 Rust 库，由 Rust 项目开发者维护。

在 OpenHarmony 中，glob 作为**基础工具库**，为编译系统和 Rust 生态提供文件路径匹配能力。该库采用零 Patch 适配策略，完全保留了原始功能。

### 核心特性

- 支持 Unix shell 通配符模式（`*.rs`, `**/*.md` 等）
- 纯 Rust 实现，跨平台兼容
- 提供灵活的匹配选项（大小写敏感、分隔符要求等）
- 高效的文件系统遍历

## OpenHarmony 适配概述

### 适配状态

| 项 | 状态 |
|----|------|
| **Patch 文件** | ❌ 无 |
| **源代码修改** | ❌ 无 |
| **构建适配** | ✅ BUILD.gn |
| **组件注册** | ✅ bundle.json |
| **文档完善** | ✅ README.OpenSource |

### 适配复杂度

⭐ **极低** - 理想的零 Patch 适配案例

**原因**:
- glob 是纯 Rust 实现的跨平台库
- 不依赖特定操作系统的 libc 函数
- OH 编译环境完全支持 Rust 2015 edition
- 仅用于基础文件匹配功能，无需 OH 特性扩展

## 文档导航

### 快速开始

| 角色 | 阅读路线 |
|------|---------|
| **开发者** | [01_Overview](01_Overview.md) → [03_Build_Integration](03_Build_Integration.md) |
| **维护者** | [02_Patches](02_Patches.md) → [06_Security](06_Security.md) |
| **审计者** | [04_Usage_in_OH](04_Usage_in_OH.md) → [05_API_Differences](05_API_Differences.md) |
| **全部** | [SUMMARY.md](SUMMARY.md) |

### 详细文档

| 文档 | 说明 | 状态 |
|------|------|------|
| [SUMMARY.md](SUMMARY.md) | 文档阅读路线建议 | ✅ |
| [01_Overview.md](01_Overview.md) | 原始库简介 | ✅ |
| [02_Patches.md](02_Patches.md) | Patch 详细分析（无 Patch） | ⏭️ 跳过 |
| [03_Build_Integration.md](03_Build_Integration.md) | OH 构建适配详解 | ✅ |
| [04_Usage_in_OH.md](04_Usage_in_OH.md) | 依赖关系与使用场景 | ✅ |
| [05_API_Differences.md](05_API_Differences.md) | API/接口差异（无差异） | ✅ |
| [06_Security.md](06_Security.md) | 安全风险分析 | ✅ |

## 工作进展

| 阶段 | 进度 |
|------|------|
| Phase 0: 信息收集 | ✅ 100% |
| Phase 1: Patch 分析 | ⏭️ 跳过（无 Patch） |
| Phase 2: 依赖关系梳理 | ✅ 100% |
| Phase 3: 文档编写 | 🔄 30% |
| Phase 4: 质量校验 | ⏳ 待开始 |

## 关键信息

### 基础信息

| 项目 | 内容 |
|------|------|
| **OH 组件名称** | rust_glob |
| **子系统** | thirdparty |
| **适配系统类型** | standard |
| **负责人** | fangting12@huawei.com |
| **上游版本** | 0.3.1 |
| **OH 版本** | 6.1 |

### 依赖关系

```mermaid
graph LR
    A[bindgen<br/>C/C++ FFI 绑定生成器] -->|使用| B[clang-sys<br/>libclang 绑定]
    B -->|文件匹配| C[glob<br/>模式匹配]
    D[OH 模块] -->|依赖| A
```

**直接依赖者**: clang-sys
**间接依赖者**: bindgen → OH 各模块

### 构建集成

```gn
# BUILD.gn
ohos_cargo_crate("lib") {
    crate_name = "glob"
    crate_type = "rlib"
    crate_root = "src/lib.rs"
    edition = "2015"
    cargo_pkg_version = "0.3.1"
}
```

## 参考链接

- [原始仓库](https://github.com/rust-lang/glob)
- [官方文档](https://docs.rs/glob/0.3.1)
- [crates.io](https://crates.io/crates/glob)
- [OpenHarmony 第三方库规范](https://gitee.com/openharmony/community/blob/master/contributing/binary-guideline/binary_library_management_guide.md)

## 反馈与贡献

如有问题或建议，请联系：
- **负责人**: fangting12@huawei.com
- **问题追踪**: [Gitee Issues](https://gitee.com/openharmony/build/issues)

---

**最后更新**: 2026-02-08
**文档版本**: 1.0.0
