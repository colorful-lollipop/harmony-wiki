# Wiki 导航

> Time Service 文档站点地图

---

## 快速导航

- [Wiki 首页](./README.md)
- [项目概览](./00_Overview.md)

---

## 核心文档

### 1. 项目概览
[00_Overview.md](./00_Overview.md)

项目定位、边界、核心能力、运行环境、关键概念。

### 2. 目录结构
[01_Directory_Structure.md](./01_Directory_Structure.md)

- 顶层目录说明
- 源码分类（framework/interfaces/services/utils）
- 模块职责划分

### 3. 架构说明
[02_Architecture.md](./02_Architecture.md)

- 组件关系图（Mermaid）
- 数据流：JS → NAPI → IPC → Service
- 线程模型
- 关键时序

### 4. N-API 参考
[03_NAPI_Reference.md](./03_NAPI_Reference.md)

- @ohos.systemTime
- @ohos.systemTimer
- @ohos.systemDateTime
- API 清单表（参数、返回值、错误码）

### 5. 内部 API
[04_Inner_API.md](./04_Inner_API.md)

- TimeServiceClient 接口
- ITimerInfo 接口
- 模块依赖方向

### 6. 构建系统
[05_GN_Targets.md](./05_GN_Targets.md)

- GN 目标列表
- 编译产物映射
- Feature Flags

### 7. 安全分析
[06_Security_Analysis.md](./06_Security_Analysis.md)

- 攻击面清单
- 权限校验点
- 可利用点分析

### 8. 问题排查
[07_Troubleshooting.md](./07_Troubleshooting.md)

- 常见问题
- 日志分析
- 调试命令

---

## 附录

### 调用链分析
[appendix/Callgraphs.md](./appendix/Callgraphs.md)

- setTime 调用链
- createTimer 调用链
- 定时器触发回调链

### 配置标志
[appendix/Config_Flags.md](./appendix/Config_Flags.md)

- time.gni 配置项
- Feature flags 说明

---

## 外部参考

- [官方 README](../README.md)
- [官方 README (中文)](../README_zh.md)
- [Bundle 配置](../bundle.json)
- [GN 配置](../time.gni)
