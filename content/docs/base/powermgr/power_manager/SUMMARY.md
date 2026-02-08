# Power Manager 模块 - 完整文档导航

本文档提供 OpenHarmony `power_manager` 模块的完整文档索引。

---

## 核心文档

### [README](README.md)
- 项目概述、覆盖范围、快速导航
- 技术栈、代码结构概览
- 更新说明与贡献指南

### [01_Overview](01_Overview.md)
- 项目定位与核心能力
- 目录结构详解
- 模块职责划分
- 依赖关系分析

### [02_Architecture](02_Architecture.md)
- 系统架构图
- 分层架构说明
- 数据流向
- 关键组件交互

---

## API 参考

### [03_NAPI_Reference](03_NAPI_Reference.md)
**Power 模块** (`@ohos.power`)
- 所有 JS/TS API 清单
- 参数、返回值、错误码
- 枚举类型定义
- 使用示例

**RunningLock 模块** (`@ohos.runningLock`)
- 运行锁 API 清单
- 类型定义与使用

### [04_Inner_API](04_Inner_API.md)
- C++ 内部 API 头文件索引
- PowerMgrClient 接口
- 回调接口定义
- 关机/休眠/唤醒相关接口

---

## 架构深入

### [05_SA_IPC](05_SA_IPC.md)
- System Ability 注册与生命周期
- ZIDL IPC 通信模式
- Proxy/Stub 实现细节
- 接口描述符

### [06_Build](06_Build.md)
- GN 构建目标清单
- Feature Flags 配置
- 编译产物清单
- 安装路径与依赖关系

---

## 安全与运维

### [07_Security](07_Security.md)
- 攻击面分析
- 信任边界
- 安全风险清单
- 修复建议

### [08_FAQ](08_FAQ.md)
- 构建问题
- 运行问题
- 调试技巧
- 常见错误排查

---

## 附录

### [appendix/Callgraphs](appendix/Callgraphs.md)
- JS → NAPI → IPC → Service 调用链
- 核心流程时序图

### [appendix/Config_Flags](appendix/Config_Flags.md)
- 所有 Feature Flags 清单
- 默认值与说明
- 编译开关

---

## 新人阅读路线

```
1. 新手入门:
   README → 01_Overview → 03_NAPI_Reference (快速浏览)

2. 开发者 (需要修改代码):
   01_Overview → 02_Architecture → 03_NAPI_Reference → 05_SA_IPC → 06_Build

3. 安全审计:
   07_Security → 05_SA_IPC (攻击面分析)

4. 构建/发布:
   06_Build → appendix/Config_Flags
```

---

## 文档变更记录

| 日期 | 版本 | 变更内容 |
|------|------|----------|
| 2026-02-06 | v1.0 | 初始版本发布 |

---

## 贡献者

- 文档生成: Sisyphus AI Agent
- 数据来源: 代码仓库静态分析
