# 阅读路线

本文档旨在帮助开发者快速了解 OpenGL ES Registry 在 OpenHarmony 中的集成与适配。请根据您的角色和需求选择相应的阅读路线。

## 开发者阅读路线

### 路线一：快速概览（5 分钟）

适合：需要快速了解该库在 OH 中的定位的开发者

| 文档 | 内容 | 时间 |
|------|------|------|
| [README.md](./README.md) | 库概览、适配特点、核心组件 | 3 分钟 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系图、典型使用场景 | 2 分钟 |

### 路线二：集成开发（15 分钟）

适合：需要在应用中集成 OpenGL ES 的开发者

| 文档 | 内容 | 时间 |
|------|------|------|
| [README.md](./README.md) | 库概览和导航 | 3 分钟 |
| [01_Overview.md](./01_Overview.md) | 原始库功能介绍 | 3 分钟 |
| [03_Build_Integration.md](./03_Build_Integration.md) | BUILD.gn 配置详解 | 4 分钟 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 使用示例和依赖关系 | 5 分钟 |

### 路线三：系统开发（30 分钟）

适合：负责图形子系统维护和适配的开发者

| 文档 | 内容 | 时间 |
|------|------|------|
| [README.md](./README.md) | 完整文档导航 | 5 分钟 |
| [01_Overview.md](./01_Overview.md) | 深入了解原始库 | 10 分钟 |
| [02_Patches.md](./02_Patches.md) | Patch 分析（理解无 Patch 设计） | 5 分钟 |
| [03_Build_Integration.md](./03_Build_Integration.md) | 构建系统深度分析 | 5 分钟 |
| [04_Usage_in_OH.md](./04_Usage_in_OH.md) | 依赖关系和集成点 | 5 分钟 |

## 按角色索引

### 应用开发者

重点关注：[04_Usage_in_OH.md](./04_Usage_in_OH.md)

了解如何：
- 在应用中引用 OpenGL ES 头文件
- 使用 NDK 接口进行 OpenGL ES 开发
- 避免常见的集成问题

### 图形库开发者

重点关注：[03_Build_Integration.md](./03_Build_Integration.md) 和 [02_Patches.md](./02_Patches.md)

了解如何：
- 正确配置 BUILD.gn 依赖
- 理解无 Patch 设计的原因
- 处理平台差异

### 系统集成工程师

全部文档都需要深入理解

重点关注：
- [03_Build_Integration.md](./03_Build_Integration.md)：构建系统适配
- [04_Usage_in_OH.md](./04_Usage_in_OH.md)：依赖关系图

## 文档更新日志

| 版本 | 日期 | 变更 |
|------|------|------|
| 1.0 | 2026-02-08 | 初始版本 |
