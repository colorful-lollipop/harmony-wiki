# Pasteboard Wiki 目录

## 入门指南

- [Wiki 说明](README.md)
- [项目概览](00_Overview.md)

## 架构与结构

- [架构说明](01_Architecture.md)
  - [系统架构图](01_Architecture.md#系统架构图)
  - [数据流图](01_Architecture.md#数据流图)
  - [线程模型](01_Architecture.md#线程模型)
  - [关键时序](01_Architecture.md#关键时序)
- [目录结构](02_Directory_Structure.md)
  - [顶层目录](02_Directory_Structure.md#顶层目录)
  - [services/](02_Directory_Structure.md#services)
  - [framework/](02_Directory_Structure.md#framework)
  - [interfaces/](02_Directory_Structure.md#interfaces)
  - [adapter/](02_Directory_Structure.md#adapter)

## 接口文档

- [N-API 参考](03_NAPI_Reference.md)
  - [模块注册](03_NAPI_Reference.md#模块注册)
  - [导出方法清单](03_NAPI_Reference.md#导出方法清单)
  - [参数校验](03_NAPI_Reference.md#参数校验)
  - [调用链](03_NAPI_Reference.md#调用链)
- [内部 API](04_Inner_API.md)
  - [PasteboardClient](04_Inner_API.md#pasteboardclient)
  - [IPC 接口](04_Inner_API.md#ipc-接口)
  - [模块依赖](04_Inner_API.md#模块依赖)

## 构建与产物

- [GN 构建](05_GN_Targets.md)
  - [构建目标](05_GN_Targets.md#构建目标)
  - [依赖关系](05_GN_Targets.md#依赖关系)
  - [编译产物](05_GN_Targets.md#编译产物)
- [附录: Feature Flags](appendix/Config_Flags.md)

## 安全与调试

- [攻击面分析](05_AttackSurface.md)
  - [攻击面总览](05_AttackSurface.md#1-攻击面总览)
  - [外部输入清单](05_AttackSurface.md#2-外部输入清单)
  - [敏感操作清单](05_AttackSurface.md#3-敏感操作清单)
  - [信任边界](05_AttackSurface.md#4-信任边界图)
  - [攻击路径](05_AttackSurface.md#5-数据流与攻击路径)
  - [高危场景](05_AttackSurface.md#6-高危攻击场景)
- [安全评审](06_Security.md)
  - [风险点清单](06_Security.md#风险点清单)
  - [修复建议](06_Security.md#修复建议)
- [问题排查](07_Troubleshooting.md)
  - [构建问题](07_Troubleshooting.md#构建问题)
  - [运行问题](07_Troubleshooting.md#运行问题)
  - [调试方法](07_Troubleshooting.md#调试方法)

## 附录

- [调用链图](appendix/Callgraphs.md)

---

## 双路线导航

### 📘 新人学习路线

快速理解项目，掌握基本使用方法：

#### 路线 1: 快速了解（30 分钟）
1. [项目概览](00_Overview.md) - 了解项目定位和核心能力
2. [目录结构](02_Directory_Structure.md) - 熟悉代码组织
3. [N-API 参考](03_NAPI_Reference.md) - 了解对外接口

#### 路线 2: 深度理解（2 小时）
1. [项目概览](00_Overview.md)
2. [架构说明](01_Architecture.md) - 理解组件关系和数据流
3. [目录结构](02_Directory_Structure.md)
4. [内部 API](04_Inner_API.md) - 理解模块接口
5. [GN 构建](05_GN_Targets.md) - 理解构建系统

### 🔒 安全研究路线

专注攻击面识别、漏洞分析和安全防护：

#### 路线 1: 快速审计（1 小时）
1. [项目概览](00_Overview.md) - 了解安全特性
2. [攻击面分析](05_AttackSurface.md) - 识别所有攻击入口
   - 外部输入清单
   - 敏感操作清单
   - 信任边界
3. [安全评审](06_Security.md) - 风险点清单

#### 路线 2: 深度安全审计（4 小时）
1. [攻击面分析](05_AttackSurface.md) - 完整攻击面梳理
2. [架构说明](01_Architecture.md) - 理解信任边界和数据流
3. [N-API 参考](03_NAPI_Reference.md) - 检查接口参数校验
4. [内部 API](04_Inner_API.md) - IPC 接口和权限检查点
5. [安全评审](06_Security.md) - 详细风险和修复建议
6. [目录结构](02_Directory_Structure.md) - 定位关键代码文件

#### 路线 3: 漏洞挖掘（持续）
1. [攻击面分析](05_AttackSurface.md) - 攻击场景分析
2. 代码审计重点关注：
   - `services/core/src/pasteboard_service.cpp:947-982` - 权限检查
   - `framework/tlv/tlv_readable.cpp` - 序列化安全
   - `interfaces/kits/napi/src/napi_*.cpp` - 参数校验
3. 模糊测试：运行现有 fuzztest
4. [GN 构建](05_GN_Targets.md) - 了解编译选项和特性开关

---

## 阅读路线（旧版）

### 路线 1: 快速了解（30 分钟）

### 路线 1: 快速了解（30 分钟）
1. [项目概览](00_Overview.md) - 了解项目定位和核心能力
2. [目录结构](02_Directory_Structure.md) - 熟悉代码组织
3. [N-API 参考](03_NAPI_Reference.md) - 了解对外接口

### 路线 2: 深度理解（2 小时）
1. [项目概览](00_Overview.md)
2. [架构说明](01_Architecture.md) - 理解组件关系和数据流
3. [目录结构](02_Directory_Structure.md)
4. [内部 API](04_Inner_API.md) - 理解模块接口
5. [GN 构建](05_GN_Targets.md) - 理解构建系统

### 路线 3: 安全审计（1 小时）
1. [架构说明](01_Architecture.md) - 理解信任边界
2. [安全评审](06_Security.md) - 全面了解安全风险
3. [N-API 参考](03_NAPI_Reference.md) - 检查接口参数校验

---

**提示**: 点击链接可直接跳转到对应文档。所有文档中的代码引用均包含文件路径和行号，可直接在代码库中定位。
