# SUMMARY

> surface_lite Wiki 站点导航

---

## 快速开始

* [Wiki 首页](./README.md)
* [项目概览](./00_Overview.md)

---

## 核心文档

### 架构与设计

* [架构说明](./01_Architecture.md)
  * 组件关系图
  * 数据流分析
  * 状态机设计
  * 线程模型

* [目录结构](./02_DirectoryStructure.md)
  * 模块职责
  * 文件组织
  * 依赖关系

### API 文档

* [对外 API (Kits)](./03_Public_API.md)
  * Surface 类接口
  * SurfaceBuffer 类接口
  * IBufferConsumerListener 接口
  * 常量与枚举
  * 错误码

* [内部 API (Innerkits)](./04_Inner_API.md)
  * SurfaceImpl 实现
  * BufferQueue 机制
  * BufferManager 单例
  * IPC 通信

### 构建与部署

* [GN 构建系统](./05_GN_Build.md)
  * Build Targets
  * 依赖关系
  * 输出产物
  * 编译命令

---

## 附录

### 安全与维护

* [安全风险评审](./06_Security.md)
  * 攻击面分析
  * 风险点清单
  * 修复建议
  * 检查局限性

* [问题排查指南](./07_Troubleshooting.md)
  * 常见问题
  * 调试方法
  * 日志分析

### 参考资料

* [调用链附录](./appendix/Callgraphs.md)
  * 关键调用流程
  * 时序图

---

## 外部链接

* [OpenHarmony 图形子系统文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/图形子系统.md)
* [窗口管理器文档](../window_window_manager_lite/README.md)

---

*最后更新: 2026-02-06*
