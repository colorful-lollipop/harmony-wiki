# 文档导航

本文档提供 ets_utils 组件的完整技术参考，包含架构设计、API 文档、构建配置和安全分析。

## 阅读路线

### 新人学习路线

#### 入门阶段 (30 分钟)

1. **[README.md](./README.md)** - 项目概述和快速开始
2. **[01_Overview.md](./01_Overview.md)** - 组件定位与核心能力
3. **[02_Architecture.md](./02_Architecture.md)** - 整体架构设计
4. **[03_CodeMap.md](./03_CodeMap.md)** - 代码地图与文件导航

#### API 阶段 (60 分钟)

5. **[03_API_js_api_module.md](./03_API_js_api_module.md)** - URL/XML/Buffer API
6. **[04_API_js_util_module.md](./04_API_js_util_module.md)** - 容器与工具 API
7. **[05_API_js_sys_module.md](./05_API_js_sys_module.md)** - 系统 API
8. **[06_API_js_concurrent_module.md](./06_API_js_concurrent_module.md)** - 并发 API

#### 进阶阶段 (45 分钟)

9. **[07_Build_Configuration.md](./07_Build_Configuration.md)** - GN 构建配置
10. **[08_Compilation_Artifacts.md](./08_Compilation_Artifacts.md)** - 编译产物
11. **[09_Security_Review.md](./09_Security_Review.md)** - 安全风险评审

### 安全研究路线

#### 快速上手 (30 分钟)

1. **[README.md](./README.md)** - 项目概述
2. **[01_Overview.md](./01_Overview.md)** - 组件定位与暴露面
3. **[03_CodeMap.md](./03_CodeMap.md)** - 核心文件定位

#### 深度分析 (90 分钟)

4. **[05_AttackSurface.md](./05_AttackSurface.md)** - 攻击面分析
5. **[09_Security_Review.md](./09_Security_Review.md)** - 安全风险评审
6. **[02_Architecture.md](./02_Architecture.md)** - 架构与数据流

#### 代码审计 (按需)

7. **[03_API_js_api_module.md](./03_API_js_api_module.md)** - API 实现细节
8. **[05_API_js_sys_module.md](./05_API_js_sys_module.md)** - 系统 API 细节
9. **[06_API_js_concurrent_module.md](./06_API_js_concurrent_module.md)** - 并发模块细节

## 完整目录

### 概览与入门

| 文档 | 描述 |
|------|------|
| [README.md](./README.md) | 项目概述、快速开始、目录结构 |
| [SUMMARY.md](./SUMMARY.md) | 本导航文档 |
| [01_Overview.md](./01_Overview.md) | 组件定位、边界、核心能力、运行环境 |
| [02_Architecture.md](./02_Architecture.md) | 组件图、数据流、线程模型、关键时序 |
| [03_CodeMap.md](./03_CodeMap.md) | 目录结构、核心文件定位、代码导航图 |

### API 参考

| 文档 | 描述 | 模块 |
|------|------|------|
| [03_API_js_api_module.md](./03_API_js_api_module.md) | URL、URI、XML、Buffer | js_api_module |
| [04_API_js_util_module.md](./04_API_js_util_module.md) | 容器、JSON、编码工具 | js_util_module |
| [05_API_js_sys_module.md](./05_API_js_sys_module.md) | 进程、定时器、控制台 | js_sys_module |
| [06_API_js_concurrent_module.md](./06_API_js_concurrent_module.md) | Worker、Taskpool | js_concurrent_module |

### 构建与部署

| 文档 | 描述 |
|------|------|
| [07_Build_Configuration.md](./07_Build_Configuration.md) | GN Targets、依赖关系 |
| [08_Compilation_Artifacts.md](./08_Compilation_Artifacts.md) | .so/.abc 产物、安装路径、加载关系 |

### 安全与分析

| 文档 | 描述 |
|------|------|
| [05_AttackSurface.md](./05_AttackSurface.md) | 攻击面分析、信任边界、输入清单 |
| [09_Security_Review.md](./09_Security_Review.md) | 安全风险评审、风险点、修复建议 |

### 附录

| 文档 | 描述 |
|------|------|
| [10_Troubleshooting.md](./10_Troubleshooting.md) | 常见问题与解决方案 |

## API 速查表

| 模块 | 命名空间 | 主要类 | JS API |
|------|---------|--------|--------|
| URL | `@ohos.url` | URL, URLSearchParams | `new URL()`, `searchParams` |
| XML | `@ohos.xml` | XmlSerializer, XmlPullParser | `XmlSerializer`, `XmlPullParser` |
| Buffer | `@ohos.buffer` | Buffer, Blob | `buffer.alloc()`, `new Blob()` |
| Util | `@ohos.util` | TextEncoder, LruBuffer | `TextEncoder`, `LruBuffer` |
| Process | `Process` | Process | `Process.pid`, `Process.kill()` |
| Worker | `@ohos.worker` | Worker | `new Worker()` |

## 版本历史

| 版本 | 日期 | 变更 |
|------|------|------|
| 1.0 | 2026-02-06 | 初始 Wiki 版本 |

---

*导航更新时间: 2026-02-06*
