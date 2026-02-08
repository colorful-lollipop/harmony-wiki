# SUMMARY - 全站导航

本文档为 `screenlock_mgr` 子系统的完整 Wiki 导航，提供新人阅读路线图。

---

## 阅读路线图

### 阶段 1：快速入门

| 顺序 | 文档 | 预计时间 | 目标 |
|------|------|----------|------|
| 1 | [README](README.md) | 2 min | 了解项目定位和文档结构 |
| 2 | [00_Overview](00_Overview.md) | 5 min | 掌握核心概念和术语 |

### 阶段 2：接口学习

| 顺序 | 文档 | 预计时间 | 目标 |
|------|------|----------|------|
| 3 | [01_NAPI_Reference](01_NAPI_Reference.md) | 15 min | 掌握 JS/N-API 接口使用方法 |

### 阶段 3：深入理解

| 顺序 | 文档 | 预计时间 | 目标 |
|------|------|----------|------|
| 4 | [02_Architecture](02_Architecture.md) | 10 min | 理解系统架构和组件关系 |
| 5 | [appendix/Callgraphs](appendix/Callgraphs.md) | 10 min | 跟踪关键调用链 |

### 阶段 4：构建与部署

| 顺序 | 文档 | 预计时间 | 目标 |
|------|------|----------|------|
| 6 | [03_GN_Build](03_GN_Build.md) | 10 min | 理解构建系统和编译产物 |

### 阶段 5：安全评审

| 顺序 | 文档 | 预计时间 | 目标 |
|------|------|----------|------|
| 7 | [04_Security_Review](04_Security_Review.md) | 15 min | 了解安全风险和最佳实践 |

---

## 文档索引

### 核心文档

| 文档 | 说明 | 关键内容 |
|------|------|----------|
| [README](README.md) | 首页 | 项目概览、快速导航 |
| [00_Overview](00_Overview.md) | 概览 | 目录结构、核心概念、运行环境 |
| [01_NAPI_Reference](01_NAPI_Reference.md) | 接口 | API 清单、参数、错误码 |
| [02_Architecture](02_Architecture.md) | 架构 | 组件图、数据流、线程模型 |
| [03_GN_Build](03_GN_Build.md) | 构建 | GN targets、编译产物 |
| [04_Security_Review](04_Security_Review.md) | 安全 | 攻击面、风险点 |

### 附录

| 文档 | 说明 |
|------|------|
| [appendix/Callgraphs](appendix/Callgraphs.md) | 关键调用链 |

---

## 术语表

| 术语 | 含义 |
|------|------|
| SA | System Ability (系统能力) |
| N-API | Native API (Node.js 原生模块接口) |
| IPC | Inter-Process Communication (进程间通信) |
| ETS | Extended TypeScript (ArkUI 声明式 UI 框架) |

---

## 相关链接

- [OpenHarmony 官方文档](https://gitee.com/openharmony/docs)
- [JS API 参考](https://gitee.com/openharmony/docs/blob/master/en/application-dev/reference/apis/js-apis-screen-lock.md)
- [官方源码仓库](https://gitee.com/openharmony/base_theme_screenlock_mgr)
