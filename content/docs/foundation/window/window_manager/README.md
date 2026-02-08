# Window Manager Wiki

## 简介

本文档是 OpenHarmony Window Manager 子系统的工程 Wiki，旨在帮助开发者快速理解项目架构、接口定义和安全特性。

## 覆盖范围

### 已覆盖内容
- 项目架构与核心组件
- 目录结构与模块职责
- N-API/JS API 接口清单
- 内部 API 与模块依赖
- GN 构建目标与编译产物
- 安全风险分析与威胁模型

### 未覆盖内容
- 详细 API 使用示例（请参考官方文档）
- 单元测试实现细节
- 历史版本变更记录

## 阅读建议

**新手上路**：
1. [项目概览](01_Overview.md) - 了解 Window Manager 定位和核心能力
2. [架构说明](02_Architecture.md) - 理解组件关系和数据流
3. [目录结构](03_Directory_Structure.md) - 熟悉代码组织

**接口开发**：
1. [N-API 参考](04_NAPI_Reference.md) - JS API 清单和调用链
2. [内部 API](05_Inner_API.md) - Native 接口使用

**构建与部署**：
1. [GN Targets](06_GN_Targets.md) - 构建目标与产物

**安全审计**：
1. [安全分析](07_Security_Analysis.md) - 攻击面和风险点

## 更新方式

本文档基于代码仓库自动生成，主要信息来源：
- `bundle.json` - 组件配置
- `BUILD.gn` - 构建配置
- 源码文件 - 接口实现

**最后更新**：2025-02-06

## 相关链接

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [Window API 参考](https://gitee.com/openharmony/docs/blob/master/en/application-dev/reference/apis/js-apis-window.md)
- [Display API 参考](https://gitee.com/openharmony/docs/blob/master/en/application-dev/reference/apis/js-apis-display.md)
