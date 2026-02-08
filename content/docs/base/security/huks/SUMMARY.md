# HUKS 文档中心

> OpenHarmony HUKS 项目 Wiki 文档索引

**生成时间**: 2025-02-06
**项目**: OpenHarmony HUKS (Hardware User Key Store)
**版本**: 4.0.2

---

## 文档说明

本文档中心提供 HUKS 项目的完整技术文档，适合新人快速了解项目架构、API 接口、构建系统和安全设计。

### 适用人群

- **新人入门**: 建议按阅读顺序从头阅读
- **应用开发者**: 重点关注 [对外 API](./03_External_API.md)
- **系统开发者**: 重点关注 [内部架构](./02_Architecture.md) 和 [内部 API](./04_Internal_API.md)
- **安全审计**: 重点关注 [安全风险评审](./07_Security_Audit.md)
- **构建维护**: 重点关注 [GN Targets](./05_GN_Targets.md) 和 [编译产物](./06_Build_Artifacts.md)

---

## 文档导航

### 快速导航

| 文档 | 内容 | 适用人群 |
|-----|------|---------|
| [项目概览](./00_Overview.md) | 项目定位、核心能力、运行环境 | 所有人 |
| [目录结构](./01_Directory_Structure.md) | 目录组织、模块职责 | 所有人 |
| [架构说明](./02_Architecture.md) | 组件图、数据流、线程模型 | 系统开发者 |
| [对外 API](./03_External_API.md) | N-API 接口、参数、错误码 | 应用开发者 |
| [内部 API](./04_Internal_API.md) | 模块接口、依赖关系 | 系统开发者 |
| [GN Targets](./05_GN_Targets.md) | 构建目标、依赖、产物 | 构建维护 |
| [编译产物](./06_Build_Artifacts.md) | 输出文件、安装路径 | 构建维护 |
| [安全风险评审](./07_Security_Audit.md) | 攻击面、信任边界、安全建议 | 安全审计 |
| [常见问题](./08_Common_Issues.md) | 构建、运行、调试问题 | 开发者 |

---

## 新人阅读路线

### 路线 1: 快速了解（30 分钟）

1. [项目概览](./00_Overview.md) - 了解 HUKS 是什么
2. [目录结构](./01_Directory_Structure.md) - 了解代码组织
3. [对外 API](./03_External_API.md) - 了解如何使用 HUKS

### 路线 2: 深入理解（2 小时）

1. [项目概览](./00_Overview.md) - 了解 HUKS 定位和边界
2. [架构说明](./02_Architecture.md) - 理解三层架构和数据流
3. [对外 API](./03_External_API.md) - 学习 N-API 接口设计
4. [内部 API](./04_Internal_API.md) - 理解模块间协作
5. [安全风险评审](./07_Security_Audit.md) - 了解安全设计

### 路线 3: 系统开发（4 小时）

1. [项目概览](./00_Overview.md) - 完整了解项目背景
2. [目录结构](./01_Directory_Structure.md) - 熟悉代码组织
3. [架构说明](./02_Architecture.md) - 深入理解架构设计
4. [GN Targets](./05_GN_Targets.md) - 掌握构建系统
5. [编译产物](./06_Build_Artifacts.md) - 了解运行时组件
6. [常见问题](./08_Common_Issues.md) - 学习问题定位方法

---

## 附录

- [关键调用链](./appendix/Callgraphs.md) - 入口到核心逻辑的完整调用链
- [配置参数](./appendix/Config_Flags.md) - 关键宏和 feature flags 说明

---

## 文档更新

本文档基于 HUKS 源码版本 4.0.2 生成。如代码有更新，建议重新扫描以下关键目录：

- `interfaces/` - 接口定义
- `services/` - 服务实现
- `frameworks/` - 框架代码

更新文档的步骤详见 [README.md](./README.md)。

---

## 相关资源

### 官方文档

- [HUKS 接口文档](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/reference/apis-universal-keystore-kit/Readme-CN.md)
- [HUKS 开发指导](https://gitcode.com/openharmony/docs/blob/master/zh-cn/application-dev/security/UniversalKeystoreKit/Readme-CN.md)

### 相关仓库

- [security_huks](https://gitcode.com/openharmony/security_huks) - HUKS 主仓库
- [security_crypto_framework](https://gitcode.com/openharmony/security_crypto_framework) - 加解密算法库框架
- [security_certificate_manager](https://gitcode.com/openharmony/security_certificate_manager) - 证书管理
