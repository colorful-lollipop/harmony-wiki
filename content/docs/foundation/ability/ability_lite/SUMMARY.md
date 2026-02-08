# SUMMARY

## ability_lite Wiki 导航

### 入门指南
- [Wiki 说明](README.md)
- [项目概览](00_Overview.md)
- [目录结构](01_Directory_Structure.md)

### 架构与实现
- [架构说明](02_Architecture.md)
  - 组件架构图
  - 数据流图
  - 线程模型
  - 生命周期时序
- [GN 构建目标](06_GN_Targets.md)
- [编译产物](07_Build_Artifacts.md)

### API 参考
- [对外 Native API](03_Native_API.md)
  - AbilityKit API
  - Want API
  - 错误码定义
- [N-API / JS API](04_NAPI_JS_API.md)
  - JS API 清单
  - 参数与错误码
  - 调用链分析
- [内部 API](05_Inner_API.md)
  - AMS 服务接口
  - 模块依赖关系

### 安全与调试
- [安全风险评估](08_Security_Assessment.md)
  - 攻击面分析
  - 信任边界
  - 风险点与修复建议
- [常见问题排查](09_Troubleshooting.md)
  - 构建问题
  - 运行时问题
  - 调试方法

---

## 新人阅读路线

### 第一天：理解项目
1. [README.md](README.md) - 了解 Wiki 结构
2. [00_Overview.md](00_Overview.md) - 理解项目定位和核心概念
3. [01_Directory_Structure.md](01_Directory_Structure.md) - 熟悉代码组织

### 第二天：掌握架构
1. [02_Architecture.md](02_Architecture.md) - 深入架构设计
2. [06_GN_Targets.md](06_GN_Targets.md) - 理解构建系统
3. [07_Build_Artifacts.md](07_Build_Artifacts.md) - 了解输出产物

### 第三天：使用接口
1. [03_Native_API.md](03_Native_API.md) - 学习 Native API
2. [04_NAPI_JS_API.md](04_NAPI_JS_API.md) - 了解 JS 绑定
3. [05_Inner_API.md](05_Inner_API.md) - 掌握内部接口

### 第四天：安全与调试
1. [08_Security_Assessment.md](08_Security_Assessment.md) - 安全注意事项
2. [09_Troubleshooting.md](09_Troubleshooting.md) - 问题排查方法

---

*最后更新: 2026-02-06*
