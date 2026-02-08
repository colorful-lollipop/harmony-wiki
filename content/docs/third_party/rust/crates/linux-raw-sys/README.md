# linux-raw-sys OpenHarmony 适配文档

## 一、库概述

**linux-raw-sys** 是 OpenHarmony 第三方库中的一个 Rust 绑定库，提供 Linux 内核用户态 API 的类型绑定。该库版本为 0.1.4，许可证为 Apache-2.0 WITH LLVM-exception OR Apache-2.0 OR MIT。

### 核心定位

该库的核心作用是为 Rust 程序提供访问 Linux 内核系统调用接口的能力。然而，由于直接进行原始系统调用是繁琐且容易出错的方式，上游社区推荐在生产环境中使用建立在该库之上的 **rustix** 库。rustix 提供了类型安全、内存安全和 I/O 安全的 Linux 系统调用封装。

### OpenHarmony 适配特点

在 OpenHarmony 生态中，linux-raw-sys 扮演着底层基础设施的角色。该库本身**不需要任何 Patch**，因为它绑定的 Linux 内核 API 在包括 OpenHarmony 在内的所有 Linux 系统上保持一致。OpenHarmony 对该库的适配主要集中在构建系统集成层面，通过 BUILD.gn 文件将其纳入 OH 的 Rust 构建体系。

## 二、文档导航

### 推荐阅读顺序

| 顺序 | 文档 | 说明 | 适合人群 |
|------|------|------|----------|
| 1 | [README.md](README.md) | 本文档，提供整体概览和导航 | 所有读者 |
| 2 | [SUMMARY.md](SUMMARY.md) | 阅读路线建议 | 需要快速定位的读者 |
| 3 | [01_Overview.md](01_Overview.md) | 原始库功能介绍 | 需要了解库背景的读者 |
| 4 | [02_Patches.md](02_Patches.md) | Patch 分析（无 Patch） | 关注 OH 定制化的读者 |
| 5 | [03_Build_Integration.md](03_Build_Integration.md) | 构建适配说明 | 构建系统和 CI 工程师 |
| 6 | [04_Usage_in_OH.md](04_Usage_in_OH.md) | OH 使用情况和依赖关系 | 集成开发者和架构师 |
| 7 | [06_Security.md](06_Security.md) | 安全风险分析 | 安全工程师 |

### 快速链接

- **上游仓库**：[https://github.com/sunfishcode/linux-raw-sys](https://github.com/sunfishcode/linux-raw-sys)
- **Crates.io**：[https://crates.io/crates/linux-raw-sys](https://crates.io/crates/linux-raw-sys)
- **API 文档**：[https://docs.rs/linux-raw-sys](https://docs.rs/linux-raw-sys)

## 三、关键特性

### 3.1 功能模块

| 模块 | 功能描述 | 默认启用 |
|------|----------|----------|
| general | Linux 通用数据类型和结构定义 | 是 |
| errno | Linux 错误码定义 | 是 |
| ioctl | 设备控制接口定义 | 是 |
| netlink | 网络配置接口定义 | 否 |

### 3.2 架构支持

该库支持多种 CPU 架构的绑定，包括：

- **32位 ARM**：arm
- **64位 ARM**：aarch64
- **32位 x86**：x86
- **64位 x86**：x86_64
- **x32**：x32（x86_64 的 32 位模式）
- **MIPS**：mips、mips64
- **PowerPC**：powerpc、powerpc64
- **RISC-V**：riscv32、riscv64
- **SPARC**：sparc、sparc64
- **s390x**：IBM Z 系列

### 3.3 与 OpenHarmony 的关系

在 OpenHarmony Rust 生态中，linux-raw-sys 处于底层位置：

```
OpenHarmony 应用/模块
        ↓
    rustix（安全封装层）
        ↓
linux-raw-sys（原始绑定）
        ↓
Linux 内核系统调用接口
```

## 四、常见问题

### Q1：为什么该库不需要 Patch？

linux-raw-sys 绑定的是 Linux 内核的用户态 API。这些 API 由 Linux 内核定义，在所有基于 Linux 内核的操作系统（包括 OpenHarmony）上保持一致。该库仅包含类型定义和常量声明，不包含任何业务逻辑或平台特定代码，因此不需要针对 OpenHarmony 进行修改。

### Q2：如何在该库的基础上添加新功能？

如果您需要添加新的绑定，建议直接向上游项目提交 PR。上游项目提供了代码生成工具（位于 gen/ 目录），可以通过更新 Linux 头文件来生成新的绑定。OH 的适配会随上游版本更新而同步。

### Q3：该库与 libc 库的关系是什么？

linux-raw-sys 和 libc 都是提供 Linux API 绑定的库，但设计目标不同：

- **libc**：提供与 C 语言库兼容的绑定，注重互操作性
- **linux-raw-sys**：提供更完整的 Linux 内核 API 绑定，包括 libc 未导出的定义

rustix 库底层使用 linux-raw-sys 来提供更现代、更安全的 Rust API。

## 五、版本信息

| 项目 | 版本 |
|------|------|
| 当前版本 | 0.1.4 |
| 上游版本 | 0.1.4 |
| Rust 版本要求 | 1.48+ |
| 最后评估日期 | 2024年 |

## 六、贡献指南

如果您发现文档中的错误或有改进建议，请通过 OpenHarmony 社区渠道反馈。对于该库的 bug 报告或功能请求，建议直接向上游项目提交。

---

**文档版本**：1.0
**最后更新**：2024年
**维护者**：OpenHarmony 第三方库工具自动生成
