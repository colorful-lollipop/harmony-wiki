# SUMMARY

hiperf 工程 Wiki - 完整目录导航

## 入门指南

- [首页](index.md) - 项目概览与快速开始
- [项目定位与核心能力](01_Overview.md) - 了解 hiperf 的定位和能力边界
- [目录结构](02_Directory_Structure.md) - 代码组织与模块职责

## 架构与实现

- [架构说明](03_Architecture.md) - 组件图、数据流、线程模型、关键时序
- [对外 API](04_Public_API.md) - C++ Inner API 详细说明
  - hiperf_client 完整 API
  - hiperf_local 完整 API
  - 使用示例
- [内部 API](05_Internal_API.md) - 内部模块接口与依赖

## 构建与产物

- [GN Targets](06_GN_Targets.md) - 构建目标梳理
- [编译产物](07_Build_Artifacts.md) - 产物清单与安装路径

## 安全与问题排查

- [安全风险评审](08_Security.md) - 攻击面与可被利用点
- [常见问题](09_Troubleshooting.md) - 构建、运行、调试问题

## 附录

- [调用链](appendix/Callgraphs.md) - 关键调用链参考
- [配置 Flags](appendix/Config_Flags.md) - 关键宏与 Feature Flags

## 新人阅读路线

### 路线 1: 快速上手（30分钟）

1. [首页](index.md)
2. [项目定位与核心能力](01_Overview.md) - 只读"核心能力"和"使用场景"
3. [对外 API](04_Public_API.md) - 只读"快速开始"和"示例代码"

### 路线 2: 深度理解（2小时）

1. [首页](index.md)
2. [项目定位与核心能力](01_Overview.md)
3. [目录结构](02_Directory_Structure.md)
4. [架构说明](03_Architecture.md)
5. [对外 API](04_Public_API.md) - 完整阅读
6. [安全风险](08_Security.md) - 了解安全边界

### 路线 3: 开发贡献（4小时）

1. 完成路线 2
2. [内部 API](05_Internal_API.md)
3. [GN Targets](06_GN_Targets.md)
4. [编译产物](07_Build_Artifacts.md)
5. [附录 - 调用链](appendix/Callgraphs.md)
6. [附录 - 配置 Flags](appendix/Config_Flags.md)

## 索引

### 按主题

**API 使用**
- [hiperf_client API](04_Public_API.md#hiperf_client)
- [hiperf_local API](04_Public_API.md#hiperf_local)
- [API 示例](04_Public_API.md#使用示例)

**构建相关**
- [构建命令](06_GN_Targets.md#构建命令)
- [Targets 列表](06_GN_Targets.md#targets-列表)
- [Feature Flags](appendix/Config_Flags.md)

**安全相关**
- [权限要求](08_Security.md#权限模型)
- [攻击面](08_Security.md#攻击面清单)
- [可被利用点](08_Security.md#可被利用点)

**调试相关**
- [日志系统](03_Architecture.md#日志系统)
- [常见问题](09_Troubleshooting.md)
- [调用链](appendix/Callgraphs.md)
