# 阅读路线建议

本文档为 OpenCL-Headers OpenHarmony Wiki 的阅读路线指南，帮助不同角色的开发者快速找到所需信息。

## 读者角色与推荐阅读路线

### 库使用者

**目标**：了解如何在 OpenHarmony 应用中使用 OpenCL-Headers

| 优先级 | 文档 | 内容概要 |
|--------|------|----------|
| 必读 | [README](README.md) | 快速入门、基本使用方式 |
| 必读 | [04_使用说明](04_Usage_in_OH.md) | 集成方式、代码示例、依赖配置 |
| 推荐 | [01_概述](01_Overview.md) | 了解库的功能和能力边界 |

### 构建系统集成者

**目标**：了解如何配置 BUILD.gn 和处理构建问题

| 优先级 | 文档 | 内容概要 |
|--------|------|----------|
| 必读 | [03_构建适配](03_Build_Integration.md) | BUILD.gn 配置、编译选项、头文件路径 |
| 参考 | [README](README.md) | 快速参考、关键配置项 |
| 参考 | [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 构建配置详细分析 |

### 适配维护者

**目标**：理解 OH 适配细节，维护和升级该库

| 优先级 | 文档 | 内容概要 |
|--------|------|----------|
| 必读 | [02_Patch 分析](02_Patches.md) | OH 适配策略、保留的变更 |
| 必读 | [03_构建适配](03_Build_Integration.md) | 构建系统适配细节 |
| 必读 | [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 完整评估报告 |
| 推荐 | [01_概述](01_Overview.md) | 库的功能定位 |

### 安全审查者

**目标**：评估库的安全风险和合规性

| 优先级 | 文档 | 内容概要 |
|--------|------|----------|
| 必读 | [02_Patch 分析](02_Patches.md) | 安全相关适配 |
| 参考 | [_work/ASSESSMENT.md](_work/ASSESSMENT.md) | 安全风险分析章节 |
| 参考 | [README](README.md) | 许可证信息 |

## 按任务分类的阅读路线

### 任务一：首次集成 OpenCL

**场景**：首次在 OpenHarmony 项目中使用 OpenCL-Headers

建议阅读顺序：

1. [README](README.md) → 了解基本概念
2. [04_使用说明](04_Usage_in_OH.md) → 获取集成代码示例
3. 参考 [03_构建适配](03_Build_Integration.md) → 配置 BUILD.gn

预计阅读时间：15-20 分钟

### 任务二：排查构建问题

**场景**：遇到头文件找不到、链接错误等问题

建议阅读顺序：

1. [03_构建适配](03_Build_Integration.md) → 核对配置
2. [README](README.md) → 确认依赖配置
3. 如有必要，查看 [_work/ASSESSMENT.md](_work/ASSESSMENT.md) → 深入理解配置

预计阅读时间：10-15 分钟

### 任务三：升级上游版本

**场景**：需要将 OpenCL-Headers 升级到上游新版本

建议阅读顺序：

1. [02_Patch 分析](02_Patches.md) → 确认保留的 OH 适配
2. [_work/ASSESSMENT.md](_work/ASSESSMENT.md) → 升级建议章节
3. [03_构建适配](03_Build_Integration.md) → 验证构建配置
4. [04_使用说明](04_Usage_in_OH.md) → 确认 API 兼容性

预计阅读时间：20-30 分钟

### 任务四：理解适配原理

**场景**：需要深入了解 OH 适配策略和实现细节

建议阅读顺序：

1. [01_概述](01_Overview.md) → 了解库的功能定位
2. [02_Patch 分析](02_Patches.md) → 理解适配策略
3. [03_构建适配](03_Build_Integration.md) → 理解构建配置
4. [_work/ASSESSMENT.md](_work/ASSESSMENT.md) → 获取完整分析

预计阅读时间：30-45 分钟

## 文档章节速查

### 快速参考表

| 需要了解的内容 | 查阅章节 |
|----------------|----------|
| 库的基本功能 | [01_概述](01_Overview.md) → 功能特性 |
| OH 适配了哪些内容 | [02_Patch 分析](02_Patches.md) |
| BUILD.gn 如何配置 | [03_构建适配](03_Build_Integration.md) |
| 如何在代码中使用 | [04_使用说明](04_Usage_in_OH.md) |
| 库的安全风险 | [_work/ASSESSMENT.md](_work/ASSESSMENT.md) → 安全风险分析 |
| 版本升级注意事项 | [_work/ASSESSMENT.md](_work/ASSESSMENT.md) → 升级建议 |

### 术语速查

| 术语 | 解释 |
|------|------|
| OpenCL | 开放计算语言，用于异构系统并行编程的开放标准 |
| ICD | Installable Client Driver，OpenCL 可安装客户端驱动 |
| 动态加载 | 运行时通过 dlopen/dlsym 加载库，而非静态链接 |
| 包装器 | Wrapper，封装底层实现提供统一接口的代码层 |

## 文档更新日志

| 版本 | 日期 | 变更说明 |
|------|------|----------|
| 1.0 | 2024-05-08 | 初始版本，创建阅读路线文档 |

---

*本文档为 OpenCL-Headers Wiki 的导航文档，建议根据实际需求选择相应的阅读路线。*
