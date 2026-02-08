# 文档导航

本文档提供 OpenHarmony Kernel Linux Patches 仓库的完整导航，帮助新人快速理解项目结构和核心能力。

## 新人阅读建议

### 入门路径（推荐顺序）

1. **[项目概览](./01_Overview.md)** - 了解项目定位和核心概念
2. **[目录结构](./02_Directory_Structure.md)** - 掌握文件组织方式
3. **[支持的板卡](./03_Supported_Boards.md)** - 确认目标平台支持情况
4. **[补丁分析](./04_Patches_Analysis.md)** - 理解补丁类型和应用方式
5. **[构建系统](./05_Build_System.md)** - 学习构建流程和配置

## 完整文档列表

### 核心文档

| 文档 | 描述 | 优先级 |
|------|------|--------|
| [项目概览](./01_Overview.md) | 项目定位、技术背景、核心能力 | ⭐⭐⭐ |
| [目录结构](./02_Directory_Structure.md) | 目录组织、模块职责 | ⭐⭐⭐ |
| [支持的板卡](./03_Supported_Boards.md) | 芯片平台、开发板列表 | ⭐⭐⭐ |
| [补丁分析](./04_Patches_Analysis.md) | 补丁类型、应用流程 | ⭐⭐⭐ |
| [构建系统](./05_Build_System.md) | GN 构建、配置选项 | ⭐⭐⭐ |
| [安全评审](./06_Security_Review.md) | 安全风险、修复建议 | ⭐⭐ |

### 附录文档

| 文档 | 描述 |
|------|------|
| [配置详情](./appendix/Config_Details.md) | 配置文件详解 |
| [补丁历史](./appendix/Patch_History.md) | 补丁变更记录 |

## 快速索引

### 按任务类型

| 任务 | 相关文档 |
|------|----------|
| 了解项目背景 | [项目概览](./01_Overview.md) |
| 添加新板卡支持 | [支持的板卡](./03_Supported_Boards.md) + [补丁分析](./04_Patches_Analysis.md) |
| 修改构建配置 | [构建系统](./05_Build_System.md) |
| 安全审计 | [安全评审](./06_Security_Review.md) |

### 按内核版本

| 版本 | 相关路径 |
|------|----------|
| Linux 4.19 | `linux-4.19/` |
| Linux 5.10 | `linux-5.10/` |
| Linux 6.6 | `linux-6.6/` |

## 相关资源

- [OpenHarmony 官方文档](https://developer.harmonyos.com)
- [OpenHarmony Gitee 仓库](https://gitee.com/openharmony)
- [Linux Kernel 官方站点](https://www.kernel.org)
