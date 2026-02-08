# Camera 工程 Wiki

## 文档说明

本文档是 OpenHarmony Camera 应用（标准系统相机）的工程 Wiki，旨在帮助开发者快速理解项目结构、架构设计、API 使用和安全风险。

## 覆盖范围

| 文档 | 内容 | 状态 |
|------|------|------|
| [概览](index.md) | 项目定位、核心能力、运行环境 | 已完成 |
| [目录结构](01_Directory_Structure.md) | 目录职责、模块划分、文件组织 | 已完成 |
| [架构说明](02_Architecture.md) | 组件图、数据流、线程模型、时序 | 已完成 |
| [对外 API](03_External_API.md) | 系统能力使用、权限、错误码 | 已完成 |
| [内部 API](04_Internal_API.md) | 模块接口、依赖方向 | 已完成 |
| [编译产物](05_Build_Artifacts.md) | HAP 构建、产物清单 | 已完成 |
| [安全风险](06_Security.md) | 攻击面、信任边界、风险点 | 已完成 |

## 更新方式

本文档基于代码仓库自动生成，结合人工审查。如需更新：

1. 修改代码后，同步更新相关 Wiki 文档
2. 提交时包含文档更新
3. 重大架构变更需在 Wiki 中记录变更历史

## 生成信息

- **生成时间**: 2026-02-05
- **代码版本**: 基于最新 master 分支
- **文档版本**: v1.0

## 快速导航

### 新人必读路线
1. [项目概览](index.md) - 了解 Camera 是什么
2. [目录结构](01_Directory_Structure.md) - 理解代码组织方式
3. [架构说明](02_Architecture.md) - 掌握整体架构
4. [对外 API](03_External_API.md) - 了解系统能力调用
5. [安全风险](06_Security.md) - 安全注意事项

### 按角色导航

**应用开发者**:
- [对外 API](03_External_API.md) - 系统 API 使用参考
- [内部 API](04_Internal_API.md) - 内部模块接口

**架构师**:
- [架构说明](02_Architecture.md) - 架构设计
- [目录结构](01_Directory_Structure.md) - 模块划分

**安全工程师**:
- [安全风险](06_Security.md) - 完整安全风险分析
- [对外 API](03_External_API.md) - 权限和调用链

**构建工程师**:
- [编译产物](05_Build_Artifacts.md) - 构建配置与产物
