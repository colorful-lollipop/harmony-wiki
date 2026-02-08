# 阅读路线建议

本文档为不同角色的读者提供针对性的阅读路径建议，帮助您快速找到所需信息。

## 面向 Native 模块开发者

如果您正在开发需要暴露 Native 能力给 JavaScript/ArkTS 的模块，建议按照以下顺序阅读：

**必读内容**：[依赖关系与使用](04_Usage_in_OH.md)

这份文档详细说明了如何在 Native 模块中包含和使用 N-API 头文件，包括构建配置示例、常见使用模式和最佳实践。它还列出了当前系统中依赖该库的所有模块，可作为开发参考。

**补充阅读**：[OH 构建适配](03_Build_Integration.md)

了解构建系统的配置细节，确保您的模块能够正确引用头文件路径。

## 面向系统架构师

如果您需要了解该库在 OpenHarmony 系统中的整体定位和依赖关系，建议阅读：

**概览**：[原始库简介](01_Overview.md)

了解 Node.js N-API 的设计理念和技术特性，以及它为何被选作 OpenHarmony 的 Native 互操作标准。

**深度分析**：[依赖关系与使用](04_Usage_in_OH.md)

查看完整的依赖图和各模块使用场景，理解该库在系统架构中的基础设施地位。

## 面向构建系统维护者

如果您负责维护 OpenHarmony 的构建系统配置，需要了解该库的构建适配细节：

**必读**：[OH 构建适配](03_Build_Integration.md）

这份文档详细说明了 BUILD.gn 的配置结构、头文件导出机制和许可证管理策略。

**补充**：[Patch 详细分析](02_Patches.md)

了解为何采用零 Patch 策略，以及这一决策的技术依据。

## 面向安全审计人员

如果您需要评估该库的安全性或进行安全审计：

**重点关注**：[Patch 详细分析](02_Patches.md)

了解是否存在 OpenHarmony 特有的代码修改，以及这些修改是否引入了新的安全风险。同时建议查阅 Node.js 官方的安全公告，了解上游版本的安全修复状态。

## 快速索引

### 我想了解...

| 问题 | 找到答案的文档 |
|------|----------------|
| 这个库做什么的？ | [原始库简介](01_Overview.md) |
| OH 对它做了什么修改？ | [Patch 详细分析](02_Patches.md) |
| 如何在我的模块中使用它？ | [依赖关系与使用](04_Usage_in_OH.md) |
| 构建系统如何配置？ | [OH 构建适配](03_Build_Integration.md) |
| 都有谁在使用这个库？ | [依赖关系与使用](04_Usage_in_OH.md) |

### 文档速查表

| 文档 | 内容概要 | 适合人群 |
|------|----------|----------|
| [README.md](./README.md) | 项目概览和快速入门 | 所有读者 |
| [01_Overview.md](./01_Overview.md) | Node.js N-API 功能介绍 | 架构师、开发者 |
| [02_Patches.md](./02_Patches.md) | OH 定制化代码分析 | 安全审计、构建维护 |
| [03_Build_Integration.md](./03_Build_Integration.md) | 构建配置详解 | 构建系统维护者 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 使用场景和依赖关系 | 开发者、架构师 |
| [ASSESSMENT.md](./_work/ASSESSMENT.md) | 完整技术评估报告 | 需要深入了解的技术人员 |

## 推荐阅读顺序

### 新手入门（30 分钟）

1. 阅读 [README.md](./README.md) 了解项目整体情况
2. 阅读 [01_Overview.md](./01_Overview.md) 理解 N-API 基本概念
3. 浏览 [04_Usage_in_OH.md](./04_Usage_in_OH.md) 查看使用示例

### 深度技术（2 小时）

完成新手入门后，继续阅读：
4. 详细研究 [03_Build_Integration.md](./03_Build_Integration.md) 理解构建适配
5. 阅读 [02_Patches.md](./02_Patches.md) 了解零 Patch 策略
6. 参考 [ASSESSMENT.md](./_work/ASSESSMENT.md) 获取完整评估信息

## 反馈与贡献

如果您发现文档中的错误或有改进建议，欢迎通过 OpenHarmony 社区渠道反馈。
