# ui_lite Wiki

## 简介

本文档是 OpenHarmony `arkui/ui_lite` 模块的工程 Wiki，提供项目架构、API 接口、构建系统和安全风险的全面分析。

## 生成信息

- **生成时间**: 2026-02-06
- **代码版本**: OpenHarmony 3.1
- **文档维护**: 随代码更新需同步更新

## 覆盖范围

### 已覆盖内容

- 项目定位与核心能力
- 目录结构与模块职责
- 架构说明（组件图、数据流、线程模型）
- 对外 C++ API 完整列表
- 内部 API 与模块依赖
- GN 构建目标梳理
- 编译产物分析
- 安全风险评审

### 未覆盖内容

- 测试代码详细分析（`test/` 目录）
- Qt 模拟器实现细节（`tools/qt/` 目录）
- 具体业务场景使用示例

## 阅读指南

### 新人阅读顺序

1. [项目概览](01_Overview.md) - 了解项目定位和边界
2. [目录结构](03_Directory_Structure.md) - 熟悉代码组织
3. [架构说明](02_Architecture.md) - 理解系统架构
4. [对外 API](04_Public_API.md) - 学习如何使用
5. [GN 构建](06_GN_Targets.md) - 了解构建系统

### 进阶阅读

- [内部 API](05_Internal_API.md) - 模块间接口
- [安全风险](08_Security.md) - 安全评审
- [编译产物](07_Build_Artifacts.md) - 输出分析
- [配置宏](appendix/Config_Flags.md) - Feature Flags

## 更新方式

1. 代码变更后，同步更新对应章节
2. 新增模块需补充到目录结构和架构说明
3. API 变更需更新接口文档
4. 安全相关修改需重新评审风险

## 相关资源

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [graphic_utils_lite](https://gitee.com/openharmony/graphic_utils)
- [window_manager_lite](https://gitee.com/openharmony/graphic_wms)
