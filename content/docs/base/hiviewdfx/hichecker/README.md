# HiChecker Wiki 文档中心

## 文档概述

本文档是 OpenHarmony DFX 子系统 HiChecker 组件的工程 Wiki，旨在帮助开发者快速理解项目架构、使用方法、编译方式及安全注意事项。

### 覆盖范围

本文档涵盖以下内容：

- 项目定位与核心能力
- 目录结构与模块职责
- 架构设计（组件图、数据流、线程模型）
- 对外 API（N-API、ETS/ANI 接口）
- 内部 API 与模块依赖
- GN 构建系统与编译产物
- 安全风险评审

### 更新方式

本文档基于代码仓库静态分析生成。当代码发生以下变更时需要同步更新：

1. 新增/删除/修改 N-API 接口
2. 修改 GN 构建配置
3. 新增安全敏感代码逻辑
4. 变更模块依赖关系

**维护责任人**: HiChecker 组件Maintainers

### 生成信息

- **生成时间**: 2026-02-06
- **代码版本**: 基于当前仓库 HEAD
- **文档版本**: 1.0

### 相关链接

- [项目 README](../README_zh.md)
- [OpenHarmony DFX 子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/DFX子系统.md)
- [hiviewdfx_hilog 组件](https://gitee.com/openharmony/hiviewdfx_hilog/blob/master/README_zh.md)
- [hiviewdfx_faultloggerd 组件](https://gitee.com/openharmony/hiviewdfx_faultloggerd/blob/master/README_zh.md)
