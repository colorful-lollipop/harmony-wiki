# graphics_effect Wiki

## 项目概述

graphics_effect 是 OpenHarmony 图形子系统的重要部件，为系统提供必需的动视效算法能力。

## 覆盖范围

本 Wiki 涵盖 graphics_effect 的以下方面：

| 文档 | 描述 |
|-----|------|
| [README](README.md) | 本文档，说明覆盖范围和更新方式 |
| [SUMMARY](SUMMARY.md) | 全站导航，新人阅读路线 |
| [01_Overview](01_Overview.md) | 项目定位、核心能力、运行环境 |
| [02_Architecture](02_Architecture.md) | 组件架构、数据流、线程模型 |
| [03_InnerAPIs](03_InnerAPIs.md) | 内部 C++ API、模块接口 |
| [04_Build](04_Build.md) | GN Targets、编译产物、安装路径 |
| [05_Security](05_Security.md) | 安全风险评审 |
| [06_Troubleshooting](06_Troubleshooting.md) | 构建/运行/调试问题 |

## 文档生成信息

- **生成时间**: 2026-02-06
- **基于代码**: graphics_effect 仓库
- **证据来源**: `BUILD.gn`, `bundle.json`, `include/*.h`, `src/*.cpp`

## 更新方式

当代码发生以下变更时，需更新对应文档：

| 变更类型 | 更新文档 |
|---------|---------|
| 新增/删除效果类型 | 01_Overview.md, 03_InnerAPIs.md |
| 架构重构 | 02_Architecture.md |
| 新增 GN Target | 04_Build.md |
| 安全相关变更 | 05_Security.md |
| 构建/编译问题修复 | 06_Troubleshooting.md |

## 阅读建议

**新人阅读顺序**:
1. [01_Overview.md](01_Overview.md) - 了解项目定位
2. [02_Architecture.md](02_Architecture.md) - 理解架构设计
3. [03_InnerAPIs.md](03_InnerAPIs.md) - 掌握核心 API
4. 根据需要查阅 [04_Build.md](04_Build.md) 或 [05_Security.md](05_Security.md)

---

*文档基于代码证据生成，所有关键结论可追溯到源码*
