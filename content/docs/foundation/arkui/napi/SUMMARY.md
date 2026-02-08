# 文档导航

## 新人阅读路线

```
建议阅读顺序:
1. 概览 → 了解项目定位和核心能力
2. 目录结构 → 熟悉代码组织方式
3. 架构说明 → 理解整体设计思路
4. N-API 接口参考 → 掌握接口使用方法
5. 构建系统 → 了解编译配置
6. 安全风险评审 → 了解安全考量
```

## 完整文档列表

### 入门指南

| 文档 | 描述 |
|------|------|
| [README](README.md) | 本 Wiki 说明和更新方式 |
| [概览](01_Overview.md) | 项目定位、核心能力、运行环境 |
| [目录结构](02_Directory_Structure.md) | 模块职责和代码组织 |

### 核心架构

| 文档 | 描述 |
|------|------|
| [架构说明](03_Architecture.md) | 组件图、数据流、线程模型、关键时序 |
| [N-API 接口参考](04_NAPI_Reference.md) | 完整 API 清单、参数、返回值、错误码 |

### 工程实践

| 文档 | 描述 |
|------|------|
| [构建系统](05_Build_System.md) | GN Targets、依赖关系、编译产物 |
| [故障排查](07_Troubleshooting.md) | 常见构建/运行/调试问题 |

### 安全与附录

| 文档 | 描述 |
|------|------|
| [安全风险评审](06_Security.md) | 攻击面、信任边界、风险修复建议 |
| [附录 A: 关键调用链](appendix/Callgraphs.md) | 入口→核心逻辑调用链 |
| [附录 B: 配置开关](appendix/Config_Flags.md) | 关键宏和 feature flags |

## 快速索引

### 按功能分类

**模块开发**：
- [N-API 接口参考](04_NAPI_Reference.md)
- [模块注册模式](04_NAPI_Reference.md#模块注册)

**异步编程**：
- [Async Work](04_NAPI_Reference.md#异步操作)
- [Promise](04_NAPI_Reference.md#promise)
- [Thread-safe Function](04_NAPI_Reference.md#线程安全函数)

**构建部署**：
- [GN 构建配置](05_Build_System.md)
- [产物清单](05_Build_System.md#编译产物)

### 按代码位置索引

| 组件 | 头文件 | 文档章节 |
|------|--------|----------|
| NativeEngine | `native_engine/native_engine.h` | 架构说明 |
| ModuleManager | `module_manager/native_module_manager.h` | 目录结构 |
| ScopeManager | `scope_manager/native_scope_manager.h` | 架构说明 |
| ReferenceManager | `reference_manager/native_reference_manager.h` | 目录结构 |

## 版本信息

| 项目 | 信息 |
|------|------|
| N-API 版本 | 8 |
| 组件版本 | 3.1 |
| 子系统 | arkui |
| 许可证 | Apache-2.0 |
