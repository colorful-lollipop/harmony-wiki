# 导航目录

本文档为 `device_board_hihope` 的工程 Wiki，提供新人快速理解项目所需的完整文档。

---

## 双路线阅读指南

根据您的身份，选择以下推荐路线：

### 🟢 新人学习路线

目标：30分钟内理解项目定位、架构和基本使用方法

| 优先级 | 章节 | 描述 | 预计时间 |
|--------|------|------|----------|
| 1 | [项目概览](00_Overview.md) | 项目定位、开发板介绍、特性对比 | 5分钟 |
| 2 | [目录结构](02_Directory_Structure.md) | 代码组织、核心文件定位 | 10分钟 |
| 3 | [开发板配置](05_Board_Configurations.md) | 选择开发板、理解硬件配置 | 10分钟 |
| 4 | [GN 构建系统](04_GN_Build.md) | 编译命令、构建配置 | 5分钟 |

### 🔴 安全研究路线

目标：快速识别攻击面、信任边界和安全风险

| 优先级 | 章节 | 描述 | 预计时间 |
|--------|------|------|----------|
| 1 | [项目概览](00_Overview.md) | 理解项目边界和暴露面 | 5分钟 |
| 2 | [安全评审](07_Security_Review.md) | 风险清单、攻击路径分析 | 15分钟 |
| 3 | [攻击面分析](07_Security_Review.md#攻击面分析) | 外部输入、信任边界 | 10分钟 |
| 4 | [目录结构](02_Directory_Structure.md) | 定位敏感文件和配置 | 10分钟 |

---

## 快速开始

- [项目概览](00_Overview.md) - 快速了解项目定位
- [开发板配置](05_Board_Configurations.md) - 选择你的开发板
- [构建配置](04_GN_Build.md) - 编译项目

---

## 核心文档

### 入门

| 章节 | 描述 | 难度 |
|------|------|------|
| [00_Overview](00_Overview.md) | 项目定位、核心能力、运行环境 | ⭐ |
| [01_Project_Scope](01_Project_Scope.md) | 项目边界、模块职责划分 | ⭐ |
| [02_Directory_Structure](02_Directory_Structure.md) | 目录结构概览 | ⭐ |

### 架构与设计

| 章节 | 描述 | 难度 |
|------|------|------|
| [03_Architecture](03_Architecture.md) | 组件图、数据流、线程模型 | ⭐⭐ |
| [04_GN_Build](04_GN_Build.md) | GN 构建系统、Targets 梳理 | ⭐⭐ |
| [05_Board_Configurations](05_Board_Configurations.md) | 各开发板详细配置 | ⭐⭐ |

### 深入

| 章节 | 描述 | 难度 |
|------|------|------|
| [06_Hardware_Drivers](06_Hardware_Drivers.md) | HDF 驱动架构、外设配置 | ⭐⭐⭐ |
| [07_Security_Review](07_Security_Review.md) | 安全风险分析与建议 | ⭐⭐⭐ |
| [08_Troubleshooting](08_Troubleshooting.md) | 常见构建/运行问题 | ⭐⭐ |

---

## 附录

- [配置参数速查](appendix/Config_Flags.md) - 关键宏与 Feature Flags
- [关键调用链](appendix/Callchains.md) - 入口→核心逻辑调用链

---

## 阅读路线图

```
新人入门路线:
    README.md → 00_Overview → 02_Directory_Structure → 05_Board_Configurations → 04_GN_Build

开发者深入路线:
    03_Architecture → 04_GN_Build → 06_Hardware_Drivers → 07_Security_Review

问题排查路线:
    08_Troubleshooting → appendix/Config_Flags → 附录/Callchains

安全研究路线:
    00_Overview → 07_Security_Review → 05_Board_Configurations → 02_Directory_Structure
```

---

## 维护指南

- **文档更新**: 修改对应 `.md` 文件后更新本文档链接
- **证据要求**: 架构/安全文档需标注代码证据路径
- **N-API**: 本仓库不包含 N-API，如需查询请访问应用层仓库
