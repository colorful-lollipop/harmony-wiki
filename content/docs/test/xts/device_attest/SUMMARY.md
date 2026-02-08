# 文档导航

本文档提供 `device_attest` Wiki 的全站导航，帮助读者快速定位所需内容。

## 目录

- [首页概览](index.md)
- [项目定位与关键概念](01_Overview.md)
- [架构说明](02_Architecture.md)
- [N-API 接口文档](03_NAPI.md)
- [内部 API 文档](04_Inner_API.md)
- [GN 构建目标与产物](05_GN_Targets.md)
- [安全风险评审](06_Security.md)
- [常见问题](07_Troubleshooting.md)
- [附录：关键调用链](appendix/Callgraphs.md)

---

## 快速导航

### 按角色分类

#### 如果你是应用开发者
1. [N-API 接口文档](03_NAPI.md) - 了解如何调用设备认证接口
2. [首页概览](index.md) - 了解模块功能

#### 如果你是系统开发者
1. [架构说明](02_Architecture.md) - 理解系统架构
2. [内部 API 文档](04_Inner_API.md) - 理解模块间接口
3. [GN 构建目标](05_GN_Targets.md) - 掌握编译系统

#### 如果你是安全审计人员
1. [安全风险评审](06_Security.md) - 全面的安全分析
2. [架构说明](02_Architecture.md) - 理解信任边界与数据流
3. [N-API 接口文档](03_NAPI.md) - 检查对外攻击面

#### 如果你是 OEM 厂商
1. [项目定位与关键概念](01_Overview.md) - 理解集成要求
2. [OEM 适配接口](04_Inner_API.md#oem-适配层) - 了解适配接口
3. [架构说明](02_Architecture.md) - 理解整体架构

---

## 文档交叉引用表

| 想了解的内容 | 推荐文档 | 相关章节 |
|-------------|---------|---------|
| 模块是做什么的 | [首页](index.md), [项目定位](01_Overview.md) | 简介、核心能力 |
| 系统架构图 | [架构说明](02_Architecture.md) | 组件架构图、数据流图 |
| JS API 怎么调用 | [N-API 接口](03_NAPI.md) | API 清单表、使用示例 |
| 接口权限要求 | [N-API 接口](03_NAPI.md), [安全评审](06_Security.md) | 权限检查、系统应用要求 |
| 模块如何编译 | [GN 构建目标](05_GN_Targets.md) | Targets 列表、依赖关系 |
| 编译产物有哪些 | [GN 构建目标](05_GN_Targets.md) | 产物清单、安装路径 |
| 安全风险点 | [安全评审](06_Security.md) | 攻击面清单、可被利用点 |
| 代码执行流程 | [附录](appendix/Callgraphs.md) | 关键调用链 |
| 遇到错误怎么办 | [常见问题](07_Troubleshooting.md) | 错误码说明、定位路径 |

---

## 术语索引

常用术语快速链接：

- [manuKey](01_Overview.md#关键术语)
- [productId](01_Overview.md#关键术语)
- [token](01_Overview.md#关键术语)
- [AttestResultInfo](03_NAPI.md#attestresultinfo-接口)
- [System Ability](02_Architecture.md#system-ability-框架)

---

## 更新日志

| 日期 | 版本 | 变更内容 |
|------|------|---------|
| 2025-02-07 | 1.1 | 增强安全评审文档：新增2个HIGH风险点详细分析、详细TLS配置、攻击面清单更新 |
| 2025-02-07 | 1.1 | 增强项目定位文档：新增四阶段认证流程详解、Mermaid时序图 |
| 2025-02-07 | 1.1 | 增强架构说明文档：新增核心业务层详细文件清单、OEM适配层安全注意 |
| 2025-02-06 | 1.0 | 初始版本，完成基础文档框架 |

---

*返回 [Wiki 首页](README.md)*
