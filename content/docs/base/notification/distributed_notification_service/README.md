# OpenHarmony 分布式通知服务 Wiki

## 文档说明

本文档为 OpenHarmony 通知子系统（Distributed Notification Service）的工程 Wiki，旨在帮助开发者快速理解项目架构、接口规范、构建流程及安全考量。

## 覆盖范围

### 已覆盖内容
- 项目定位与核心能力
- 系统架构与组件关系
- N-API 接口规范（JS/TS API）
- Inner API 接口定义（C++ 模块接口）
- GN 构建系统与 Targets
- 编译产物与运行时加载关系
- 安全风险评估

### 未覆盖内容
- 测试相关代码与用例（按规范忽略）
- 详细的 fuzzing/benchmark 测试配置
- 特定设备适配细节

## 文档更新

**最后更新**: 2026-02-06

**源码版本**: 基于 OpenHarmony master 分支

**更新方式**:
1. 代码变更后主动更新相关章节
- 重大架构调整时重新评审所有章节
- 建议在 PR 中同步更新 Wiki

## 快速导航

| 主题 | 文档 | 说明 |
|------|------|------|
| 项目概览 | [01_Overview.md](./01_Overview.md) | 项目定位、核心能力、运行环境 |
| 架构设计 | [02_Architecture.md](./02_Architecture.md) | 组件图、数据流、线程模型 |
| API 参考 | [03_NAPI_Reference.md](./03_NAPI_Reference.md) | N-API 接口清单与使用 |
| 构建指南 | [04_Build_Guide.md](./04_Build_Guide.md) | GN Targets 与编译产物 |
| 安全评审 | [05_Security_Review.md](./05_Security_Review.md) | 攻击面与风险评估 |

## 相关链接

- [OpenHarmony 通知子系统源码](https://gitee.com/openharmony/notification_distributed_notification_service)
- [SystemUI 系统应用](https://gitee.com/openharmony/applications_systemui)
- [Ability Runtime](https://gitee.com/openharmony/ability_ability_runtime)
