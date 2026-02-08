# Syscap Codec Wiki

## 简介

本 Wiki 是 OpenHarmony `syscap_codec` 项目的工程文档，旨在帮助开发者快速理解项目结构、架构设计、接口定义和安全风险。

## 项目概述

**syscap_codec** 是 OpenHarmony 的系统能力（System Capability）编解码工具，主要用于：

- **PCID** (Product Compatibility ID) 的编码与解码
- **RPCID** (Required Product Compatibility ID) 的编码与解码
- 设备系统能力与应用需求的兼容性检查

## 适用对象

- 新加入项目的开发人员
- 需要集成 syscap_codec 的 IDE/工具开发者
- 进行安全审计的工程师
- 维护构建系统的工程师

## 文档导航

### 入门指南
1. [项目概览](00_Overview.md) - 了解项目定位、核心能力和运行环境
2. [目录结构](01_Directory_Structure.md) - 熟悉代码组织方式

### 技术深入
3. [架构说明](02_Architecture.md) - 组件图、数据流、线程模型
4. [N-API 接口](03_NAPI_Interface.md) - JS API 面详细说明
5. [内部 API](04_Inner_API.md) - 模块接口和依赖关系

### 工程实践
6. [GN 构建目标](05_GN_Targets.md) - 编译产物和构建配置
7. [安全风险分析](06_Security_Analysis.md) - 攻击面和风险点
8. [常见问题](07_Troubleshooting.md) - 构建/运行/调试问题

### 附录
- [调用链附录](appendix/Callgraphs.md) - 关键调用链详情
- [配置标志附录](appendix/Config_Flags.md) - 关键宏和 feature flags

## 更新记录

| 日期 | 版本 | 说明 |
|------|------|------|
| 2025-02-06 | v1.0 | 初始版本，完成基础架构文档和安全风险分析 |

## 覆盖范围

### 已覆盖内容
- ✅ 项目定位与核心能力
- ✅ 目录结构与模块职责
- ✅ 架构设计（组件图、数据流、线程模型）
- ✅ N-API 和 ANI 接口详细说明
- ✅ 内部 C/C++ API 说明
- ✅ GN 构建目标和编译产物
- ✅ 安全风险分析（10个风险点）
- ✅ 常见问题与调试方法
- ✅ 关键调用链详情
- ✅ 配置标志和宏定义

### 未覆盖内容
- ❌ Python 工具脚本详细说明（非运行时组件）
- ❌ 测试代码说明（按约束忽略）
- ❌ 性能基准测试数据
- ❌ 历史变更记录（CHANGELOG）

## 如何更新本文档

本文档基于代码证据生成，更新时请：

1. 确保代码证据准确（文件路径 + 行号）
2. 在 `_work/NOTES.md` 中记录新发现
3. 更新相关文档后修改 `SUMMARY.md`
4. 在更新记录中添加条目

## 相关资源

- [OpenHarmony 官方文档](https://gitee.com/openharmony)
- [README_ZH.md](../README_ZH.md) - 项目中文README
- [bundle.json](../bundle.json) - 组件配置

---

*本文档由工程 Agent 自动生成，如有疑问请参考代码原始文件。*
