# graphic_surface Wiki

本文档集提供 `graphic_surface` 组件的完整工程文档。

## 文档概述

本 Wiki 为新人提供理解 graphic_surface 项目所需的完整信息，包括：
- 项目定位与核心能力
- 模块职责与目录结构
- 架构设计与数据流
- 对外/内部 API 说明
- GN 构建系统与编译产物
- 安全风险分析
- 常见问题与调试方法

## 快速导航

**新人推荐阅读顺序**：
1. [概览](00_Overview.md) - 了解项目定位、核心能力、运行环境
2. [目录结构与模块职责](01_Directory_Structure.md) - 熟悉代码组织结构
3. [架构说明](02_Architecture.md) - 理解组件设计、数据流、线程模型
4. [内部 API](04_Internal_API.md) - 学习模块间接口与依赖关系
5. [GN Targets 与编译产物](05_GN_Targets.md) - 了解构建系统
6. [安全风险评审](07_Security_Review.md) - 认识潜在安全风险

**参考文档**：
- [对外 API](03_External_API.md) - Native C/C++ API 参考
- [编译产物](06_Build_Artifacts.md) - 构建输出清单
- [常见问题](08_Troubleshooting.md) - 调试与问题定位

**附录**：
- [关键调用链](appendix/Callgraphs.md) - 重要代码执行路径
- [配置宏与 Feature Flags](appendix/Config_Flags.md) - 编译开关说明

## 文档范围

### 已覆盖
- ✅ 项目架构与模块设计
- ✅ Native C/C++ API（external_window.h, native_buffer.h, surface.h 等）
- ✅ 内部模块接口（Surface, BufferQueue, SyncFence, BufferHandle）
- ✅ IPC 机制（OHOS IPC, IBufferProducer, IBufferConsumer）
- ✅ GN 构建系统
- ✅ 安全风险分析

### 未覆盖
- ❌ JavaScript/ArkTS API - 本模块不直接提供 JS 绑定，通过高层框架（如 @ohos.window）间接使用
- ❌ 测试代码与测试框架 - 单元测试、fuzz 测试、系统测试
- ❌ 依赖组件的详细实现 - 仅列出接口与依赖关系

## 文档维护

### 生成时间
- 2025-02-06 16:09

### 如何更新文档
1. **代码变更后**：检查对应章节的证据（文件路径、符号、代码片段）是否仍然有效
2. **新增 API**：在 `03_External_API.md` 或 `04_Internal_API.md` 中补充
3. **架构变更**：更新 `02_Architecture.md` 的组件图和调用链
4. **构建系统变更**：更新 `05_GN_Targets.md` 和 `06_Build_Artifacts.md`
5. **安全问题修复**：更新 `07_Security_Review.md` 的风险状态

### 证据原则
所有关键结论必须提供代码证据：
- 文件路径（必要时含行号）
- 关键符号名（函数/类/宏/target）
- 最小必要代码片段或调用链描述

无法确认的内容必须标注 `TODO(需确认)`。

## 项目基本信息

| 属性 | 值 |
|------|-----|
| 组件名称 | @ohos/graphic_surface |
| 版本 | 4.1 |
| 子系统 | graphic（图形子系统） |
| 主要功能 | 管理和传递图形与媒体的共享内存 |
| 编译产物 | surface.so, sync_fence.so, buffer_handle.so |
| 依赖组件 | access_token, ipc, hilog, hitrace, hisysevent, samgr 等 |
| 适配系统类型 | standard（标准系统） |

## 核心概念

### Surface
Surface 是图形和媒体共享内存的管理抽象，使用生产者-消费者模式：
- **生产者**（如 UI 组件）：从 Free 队列获取 Buffer → 绘制内容 → 放入 Dirty 队列
- **消费者**（如 WMS 组件）：从 Dirty 队列获取 Buffer → 合成显示 → 放回 Free 队列

### 跨进程传输
- **IPC 层**：传输 BufferHandle 等控制结构（有拷贝）
- **共享内存层**：传输图形/媒体数据（零拷贝）

### 重要提示
1. 共享内存管理在首次创建 Surface 的进程中，进程异常且未回收会导致严重内存泄漏
2. 不建议在小内存传输场景使用，会导致内存碎片化

## 相关资源

- **OpenHarmony 文档**：[图形子系统](https://gitee.com/openharmony/docs/blob/master/zh-cn/readme/图形子系统.md)
- **相关组件**：[window_window_manager](https://gitee.com/openharmony/window_window_manager)
