# libloading Wiki

## 库概览

**libloading** 是一个 Rust 语言编写的动态链接库加载库，为 Rust 提供了跨平台的动态库加载能力。该库通过 `dlopen`（Unix/Linux/macOS）和 `LoadLibrary`（Windows）系统调用实现动态库加载，并提供了比原生 API 更强的内存安全性保证。

在 OpenHarmony 生态系统中，libloading 作为 Rust 基础设施组件，被集成到 `third_party/rust/crates` 目录中，为 OpenHarmony 的 Rust 生态提供动态库加载支持。

## OpenHarmony 适配概述

libloading 在 OpenHarmony 中的集成相对简单，主要特点如下：

1. **无 Patch 状态**：该库为原生移植，未应用任何 OpenHarmony 特定 Patch
2. **标准适配**：通过标准的 Rust cfg-if 机制实现平台条件编译
3. **基础构建**：使用 OpenHarmony 的 `ohos_cargo_crate` 模板进行构建集成

## 文档导航

### 快速入门
- [README.md](README.md) - 快速了解 libloading 在 OH 中的定位
- [SUMMARY.md](SUMMARY.md) - 完整阅读路线

### 核心内容
- [01_Overview.md](01_Overview.md) - 原始库功能介绍
- [02_Patches.md](02_Patches.md) - Patch 分析（本库无 Patch）
- [03_Build_Integration.md](03_Build_Integration.md) - 构建适配详情
- [04_Usage_in_OH.md](04_Usage_in_OH.md) - OH 使用情况
- [05_API_Differences.md](05_API_Differences.md) - API 差异分析
- [06_Security.md](06_Security.md) - 安全风险分析

### 工作文件
- [_work/ASSESSMENT.md](_work/ASSESSMENT.md) - 项目评估结果
- [_work/NOTES.md](_work/NOTES.md) - 分析过程记录
- [_work/PLAN.md](_work/PLAN.md) - 任务进度追踪

## 版本信息

| 属性 | 值 |
|------|-----|
| 上游版本 | 0.7.4 |
| OH 包版本 | 6.1 |
| 上游地址 | https://github.com/nagisa/rust_libloading |
| 许可证 | MIT / ISC |
| OH 组件名 | @ohos/rust_libloading |
| 所属子系统 | thirdparty |

## 关键特性

1. **跨平台支持**：Linux、macOS、Windows 等多平台动态库加载
2. **内存安全**：相比原生 C API 提供更强的安全保障
3. **零依赖**：除 cfg-if 外无外部依赖
4. **Rust 2015 Edition**：兼容较老的 Rust 工具链版本（1.40.0+）

## 相关资源

- 上游文档：https://docs.rs/libloading/
- 上游仓库：https://github.com/nagisa/rust_libloading/
- OpenHarmony Rust 基础设施：[链接待补充]
