# Neural Network Runtime Wiki

## 简介

本 Wiki 是 OpenHarmony Neural Network Runtime (NNRt) 的工程文档，旨在帮助开发者快速理解项目架构、API 接口、构建系统和安全风险。

## 生成信息

- **生成时间**: 2025-02-06
- **代码版本**: 4.0
- **仓库路径**: `foundation/ai/neural_network_runtime`

## 覆盖范围

### 已覆盖内容

- [x] 项目概览与定位
- [x] 架构设计与数据流
- [x] 目录结构与模块职责
- [x] 对外 Native API (C 接口)
- [x] 内部 API 与接口稳定性
- [x] GN 构建目标与编译产物
- [x] 安全风险评审
- [x] 常见问题与调试

### 未覆盖内容

- [ ] N-API (JavaScript API) - 本项目为纯 Native 库，无 N-API 实现
- [ ] 测试代码详细分析 (按约束忽略)
- [ ] 具体芯片驱动实现细节

## 阅读指南

### 新人阅读顺序

1. [首页/概览](index.md) - 了解项目定位和核心能力
2. [架构说明](02_Architecture.md) - 理解系统架构和数据流
3. [目录结构](03_Directory_Structure.md) - 熟悉代码组织
4. [对外 API](04_Native_API.md) - 掌握 C API 使用方法
5. [GN 构建](06_GN_Targets.md) - 了解构建系统
6. [安全评审](08_Security_Review.md) - 了解安全风险

### 快速参考

- [API 速查](appendix/API_Quick_Reference.md)
- [错误码对照](appendix/Error_Codes.md)
- [算子列表](appendix/Operators.md)

## 文档更新

当代码发生变更时，需要同步更新以下文档：

1. 新增/修改 API → 更新 `04_Native_API.md`
2. 新增算子 → 更新 `appendix/Operators.md`
3. 修改构建配置 → 更新 `06_GN_Targets.md`
4. 新增安全风险 → 更新 `08_Security_Review.md`

## 相关链接

- [官方文档](https://gitee.com/openharmony/docs/tree/master/zh-cn/application-dev/reference/apis-neural-network-runtime-kit)
- [HDI 接口定义](https://gitee.com/openharmony/drivers_interface/tree/master/nnrt)
- [MindSpore Lite](https://gitee.com/openharmony/third_party_mindspore)
