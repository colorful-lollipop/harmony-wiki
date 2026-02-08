# SUMMARY - 全局导航

## 文档结构
本文档集提供 graphic_surface 组件的完整工程文档，按模块和主题组织。

## 新人推荐阅读路径

### 快速入门（30分钟）
1. **[README](README.md)** - 了解项目范围、生成时间、如何更新文档
2. **[概览](00_Overview.md)** - 项目定位、核心能力、运行环境
3. **[目录结构与模块职责](01_Directory_Structure.md)** - 熟悉代码组织

### 深入理解（2小时）
4. **[架构说明](02_Architecture.md)** - 组件设计、数据流、线程模型
5. **[对外 API](03_External_API.md)** - Native C/C++ API 参考
6. **[内部 API](04_Internal_API.md)** - 模块间接口与依赖

### 实践应用（按需查阅）
7. **[GN Targets 与编译产物](05_GN_Targets.md)** - 构建系统与依赖关系
8. **[编译产物](06_Build_Artifacts.md)** - 构建输出清单
9. **[安全风险评审](07_Security_Review.md)** - 安全风险与修复建议
10. **[常见问题](08_Troubleshooting.md)** - 故障排查与调试方法

### 附录参考（按需查阅）
- **[关键调用链](appendix/Callgraphs.md)** - 重要代码执行路径
- **[配置宏与 Feature Flags](appendix/Config_Flags.md)** - 编译开关说明

## 文档导航

### 核心文档

| 文档 | 页数 | 核心内容 | 难度 |
|--------|--------|----------|--------|
| [README](README.md) | 1 | Wiki 使用指南、文档范围 | ⭐ |
| [00_Overview](00_Overview.md) | - | 项目定位、核心能力、运行环境 | ⭐⭐ |
| [01_Directory_Structure](01_Directory_Structure.md) | - | 目录树、模块职责、依赖关系 | ⭐⭐ |
| [02_Architecture](02_Architecture.md) | - | 组件设计、数据流、线程模型 | ⭐⭐⭐ |
| [03_External_API](03_External_API.md) | - | Native C/C++ API 参考 | ⭐⭐⭐ |
| [04_Internal_API](04_Internal_API.md) | - | 模块间接口、稳定性标注 | ⭐⭐⭐⭐ |
| [05_GN_Targets](05_GN_Targets.md) | - | GN 构建系统、Target 依赖 | ⭐⭐⭐ |
| [06_Build_Artifacts](06_Build_Artifacts.md) | - | 编译产物、安装路径 | ⭐⭐ |
| [07_Security_Review](07_Security_Review.md) | - | 安全风险、攻击面、修复建议 | ⭐⭐⭐⭐ |
| [08_Troubleshooting](08_Troubleshooting.md) | - | 常见问题、调试方法 | ⭐⭐ |

### 附录文档

| 文档 | 内容 | 用途 |
|--------|--------|------|
| [appendix/Callgraphs](appendix/Callgraphs.md) | 关键函数调用链 | 理解代码执行路径 |
| [appendix/Config_Flags](appendix/Config_Flags.md) | Feature Flags | 控制编译特性 |

## 按主题导航

### 了解项目
- **[概览](00_Overview.md)** - 什么是 graphic_surface？
- **[目录结构与模块职责](01_Directory_Structure.md)** - 代码如何组织？
- **[README](README.md)** - Wiki 如何使用？

### 理解架构
- **[架构说明](02_Architecture.md)** - 组件如何协作？
- **[附录 - 关键调用链](appendix/Callgraphs.md)** - 代码如何执行？

### 使用 API
- **[对外 API](03_External_API.md)** - 如何使用 Surface？
- **[内部 API](04_Internal_API.md)** - 模块间如何交互？
- **[附录 - 配置宏与 Feature Flags](appendix/Config_Flags.md)** - 如何配置特性？

### 构建与部署
- **[GN Targets 与编译产物](05_GN_Targets.md)** - 如何编译？
- **[编译产物](06_Build_Artifacts.md)** - 产物是什么？

### 安全与维护
- **[安全风险评审](07_Security_Review.md)** - 有哪些安全风险？
- **[常见问题](08_Troubleshooting.md)** - 如何调试？

## 快速索引

### 关键概念
| 概念 | 文档 | 章节 |
|--------|--------|------|
| Surface | [概览](00_Overview.md) | 核心概念 |
| 生产者-消费者模式 | [架构说明](02_Architecture.md) | Buffer 轮转流程 |
| Buffer 队列（Free/Dirty） | [架构说明](02_Architecture.md) | BufferQueue 实现 |
| IPC 机制 | [架构说明](02_Architecture.md) | OHOS IPC 组件 |
| SyncFence | [概览](00_Overview.md) | 同步栅栏机制 |
| BufferHandle | [对外 API](03_External_API.md) | 跨进程传输 |
| Native Window API | [对外 API](03_External_API.md) | C API 使用 |
| PID 访问控制 | [安全风险评审](07_Security_Review.md) | 权限检查 |

### API 速查
| API 类型 | 头文件 | 文档 | 示例 |
|---------|---------|--------|------|
| Native Window C API | `external_window.h` | [对外 API](03_External_API.md) | 示例 1：生产者 |
| Native Buffer C API | `native_buffer.h` | [对外 API](03_External_API.md) | 示例 1：生产者 |
| Surface C++ API | `surface.h` | [对外 API](03_External_API.md) | 示例 2：消费者 |
| IPC Producer 接口 | `ibuffer_producer.h` | [对外 API](03_External_API.md) | IPC 方法列表 |
| Consumer Surface 接口 | `iconsumer_surface.h` | [对外 API](03_External_API.md) | 示例 2：消费者 |

### 错误码速查
| 错误码 | 含义 | 文档 |
|---------|------|--------|
| GSERROR_OK | 成功 | [对外 API - 错误码](03_External_API.md) |
| GSERROR_NO_BUFFER | 无可用 Buffer | [对外 API - 错误码](03_External_API.md) |
| GSERROR_NO_CONSUMER | 无消费者 | [对外 API - 错误码](03_External_API.md) |
| GSERROR_CONSUMER_IS_CONNECTED | 消费者已连接 | [对外 API - 错误码](03_External_API.md) |
| GSERROR_BUFFER_STATE_INVALID | Buffer 状态无效 | [对外 API - 错误码](03_External_API.md) |
| GSERROR_NO_PERMISSION | 权限不足 | [安全风险评审](07_Security_Review.md) |

### 构建相关
| 主题 | 文档 | 关键信息 |
|------|--------|---------|
| GN Targets | [GN Targets 与编译产物](05_GN_Targets.md) | surface, sync_fence, buffer_handle |
| 编译产物 | [编译产物](06_Build_Artifacts.md) | .so/.a 文件清单 |
| 依赖组件 | [GN Targets 与编译产物](05_GN_Targets.md) | c_utils, hilog, ipc 等 |
| Feature Flags | [附录 - Config_Flags](appendix/Config_Flags.md) | TV metadata, AI scheduling |

### 安全相关
| 风险类型 | 文档 | 关键发现 |
|---------|--------|---------|
| 内存泄漏 | [安全风险评审](07_Security_Review.md) | 进程异常且未回收 |
| PID 欺骗 | [安全风险评审](07_Security_Review.md) | GetCallingPid() 绕过 |
| 沙箱隔离 | [安全风险评审](07_Security_Review.md) | fd passing 安全性 |
| HEBC 白名单 | [安全风险评审](07_Security_Review.md) | JSON 配置注入 |
| Magic Number 篡改 | [安全风险评审](07_Security_Review.md) | 内存破坏检测 |

## 文档版本

### 生成时间
- 2025-02-06 16:09

### 证据截止时间
- 基于 2025-02-06 的代码快照

### 更新建议
- **代码变更后**：检查相关章节的证据（文件路径、符号、代码片段）
- **新增 API**：在 [对外 API](03_External_API.md) 或 [内部 API](04_Internal_API.md) 中补充
- **架构变更**：更新 [架构说明](02_Architecture.md) 的组件图和调用链
- **安全问题修复**：更新 [安全风险评审](07_Security_Review.md) 的风险状态

## 联系与反馈

### 文档问题
如发现文档错误或需要补充：
1. 检查 [工作笔记](wiki/_work/NOTES.md) 中是否已有记录
2. 如无，在 issue 中提出，附带：
   - 问题描述
   - 期望内容
   - 相关证据（如有）

### 代码问题
如发现代码问题：
1. 在 OpenHarmony 代码仓提 issue
2. 在 issue 中引用相关文档章节

## 相关资源

### OpenHarmony 资源
- [图形子系统文档](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/图形子系统.md)
- [Window Manager 组件](https://gitee.com/openharmony/window_window_manager)
- [OpenHarmony 官方文档](https://docs.openharmony.cn/)

### 代码仓
- [graphic_surface 代码仓](https://gitee.com/openharmony/graphic_surface)
- [OpenHarmony 主代码仓](https://gitee.com/openharmony)

## 术语表

| 术语 | 英文 | 说明 |
|--------|------|------|
| Surface | Surface | 图形 Surface，用于管理共享内存 Buffer |
| Buffer | Buffer | 图形数据存储单元，指向共享内存 |
| BufferHandle | Buffer Handle | 跨进程传输的句柄，包含 fd 等元数据 |
| 生产者 | Producer | 创建/修改 Buffer 的组件（如 UI 框架） |
| 消费者 | Consumer | 读取/使用 Buffer 的组件（如 WMS） |
| Free 队列 | Free Queue | 可用 Buffer 列表 |
| Dirty 队列 | Dirty Queue | 待消费 Buffer 列表 |
| SyncFence | Sync Fence | 同步栅栏，基于 Linux dma_fence |
| IPC | IPC | 进程间通信 |
| OHOS IPC | OHOS IPC | OpenHarmony IPC 框架 |
| Binder | Binder | Android 风格 IPC 机制 |
| GN | GN | Google Ninja 构建系统 |
| BUILD.gn | BUILD.gn | GN 构建脚本 |
| HEBC | HEBC | High Efficiency Buffer Cache |
| PID | PID | Process ID，进程标识符 |
| fd | fd | File Descriptor，文件描述符 |

## 反馈路径

本文档由 AI 自动生成，如发现：
- **内容错误**：请引用代码证据提出修正
- **章节缺失**：请说明需要补充的内容
- **链接失效**：请指明失效的链接和目标

---

**祝阅读愉快！**
