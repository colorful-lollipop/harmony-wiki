# OpenHarmony LiteOS-M Kernel Wiki

## 概述

本文档为 OpenHarmony LiteOS-M 内核的工程 Wiki，提供全面的技术参考。

## 项目定位

**LiteOS-M** 是 OpenHarmony 的轻量级操作系统内核，专为 IoT（物联网）领域设计。

- **代码特点**: 小体积、低功耗、高性能
- **语言支持**: C 和 C++（README.md:69）
- **架构支持**: ARM32, ARM64, RISC-V, C-Sky, Xtensa

## 文档覆盖范围

| 文档 | 说明 |
|------|------|
| [README](README.md) | 本文档，说明 Wiki 覆盖范围与更新方式 |
| [SUMMARY](SUMMARY.md) | 全站导航与新人阅读顺序 |
| [01_Overview](01_Overview.md) | 项目定位、边界、核心能力 |
| [02_Directory_Structure](02_Directory_Structure.md) | 目录结构与模块职责 |
| [03_Architecture](03_Architecture.md) | 架构说明、组件图、数据流 |
| [04_Kernel_API](04_Kernel_API.md) | 内核 API（LOS_* 系列） |
| [05_KAL](05_KAL.md) | 内核抽象层（CMSIS/POSIX） |
| [06_Components](06_Components.md) | 可选组件说明 |
| [07_Build](07_Build.md) | GN Targets 与编译产物 |
| [08_Security](08_Security.md) | 安全风险评审 |
| [09_FAQ](09_FAQ.md) | 常见问题与调试 |

## 不包含内容

- 测试相关代码（`testsuites/` 等）
- N-API / JS API（本项目是内核，不提供用户态 API）

## 更新方式

1. 代码变更后，同步更新相关 Wiki 章节
2. 新增模块时，创建对应的 Wiki 页面
3. 安全问题时，在 08_Security.md 中更新

## 生成时间

2026-02-06
