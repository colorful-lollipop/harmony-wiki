# OpenHarmony Form Fmk Wiki

> 本 Wiki 由工程 Wiki 生成 Agent 自动生成
> 项目路径：`foundation/ability/form_fwk`
> 组件名：`@ohos/form_fwk`

## 项目简介

**Form Framework (form_fwk)** 是 OpenHarmony 元能力子系统的核心组件，负责管理系统中**卡片（Form/Widget）**的全生命周期。

卡片是一种界面展示形式，可以将应用的重要信息或操作前置到卡片，达到服务直达的目的。常用于嵌入到系统应用中显示，支持拉起页面、发送消息等交互功能。

## 文档覆盖范围

本 Wiki 文档覆盖 OpenHarmony `form_fwk`（卡片管理框架）的完整工程信息，包括：

- **项目概述**：定位、核心能力、运行环境
- **架构设计**：组件图、数据流、线程模型
- **N-API 参考**：JS 接口、参数、错误码
- **Inner API**：模块接口、依赖方向
- **GN 构建**：Targets、编译产物
- **安全评审**：攻击面、风险点

## 受众指南

| 受众 | 推荐阅读 | 目标 |
|------|---------|------|
| **新人开发者** | [00_Overview](00_Overview.md) → [01_Project_Overview](01_Project_Overview.md) → [02_Architecture](02_Architecture.md) | 5分钟理解项目，30分钟理解架构 |
| **安全研究员** | [00_Overview](00_Overview.md) → [05_AttackSurface](05_AttackSurface.md) → [07_Security_Review](07_Security_Review.md) | 快速识别攻击面，定位安全风险 |
| **系统开发者** | [02_Architecture](02_Architecture.md) → [04_Inner_API](04_Inner_API.md) → [05_GN_Targets](05_GN_Targets.md) | 理解实现细节，进行二次开发 |

## 未覆盖范围

- 测试代码（test/ 目录）
- 第三方依赖的内部实现细节

## 更新方式

当代码仓库发生以下变更时，需同步更新 Wiki：

1. 新增/删除 N-API 接口
2. 新增/删除 Inner API 接口
3. 新增/删除 GN Targets
4. 安全相关的代码变更

## 生成信息

- **仓库**：`foundation/ability/form_fwk`
- **版本**：3.1
- **生成时间**：2026-02-06
- **最后更新**：2026-02-06

## 相关链接

- [OpenHarmony Form 开发指南](https://gitee.com/openharmony/docs/blob/master/zh-cn/application-dev/form/Readme-CN.md)
- [接口 SDK (interface_sdk-js)](https://gitee.com/openharmony/interface_sdk-js)
- [Ability Runtime](https://gitee.com/openharmony/ability_ability_runtime)
