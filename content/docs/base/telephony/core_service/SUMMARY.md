# Wiki 导航

## 入门指南

- [Wiki 首页](./README.md)
- [项目概览](./00_Overview.md)
- [目录结构](./01_Directory_Structure.md)

## 架构与接口

- [架构设计](./02_Architecture.md)
  - 组件架构图
  - 数据流图
  - 线程模型
  - 关键时序
- [N-API 接口（JS API）](./03_NAPI_API.md)
  - SIM 模块 API
  - Radio 模块 API
  - eSIM 模块 API
  - VCard 模块 API
- [内部 API（Inner Kits）](./04_Inner_API.md)
  - CoreServiceClient
  - TelRilManager 接口
  - 回调接口定义

## 构建与产物

- [GN 构建系统](./05_GN_Build.md)
  - 构建目标列表
  - 依赖关系
  - 编译产物
  - Feature Flags

## 安全与调试

- [安全风险评审](./06_Security.md)
  - 攻击面分析
  - 信任边界
  - 风险点清单
  - 修复建议
- [常见问题](./07_FAQ.md)
  - 构建问题
  - 运行时问题
  - 调试方法

## 附录

- [关键调用链](./appendix/Callgraphs.md)
- [配置与开关](./appendix/Config_Flags.md)
- [错误码参考](./appendix/Error_Codes.md)
