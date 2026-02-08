# 项目概览

## 项目定位

N-API（Native API）组件是 OpenHarmony ArkUI 框架的核心组成部分，提供基于 Node.js N-API 规范开发的原生模块扩展开发框架。

**核心价值**：
- 实现 JS 与 C/C++ 代码的互相访问
- 封装 IO、CPU 密集型、OS 底层等能力并对外暴露 JS 接口
- 提供稳定的 ABI 接口，隔离 JS 引擎变化对 native 模块的影响

**适用场景**：
- 网络通信（串口、网络 sockets）
- 多媒体解码
- 传感器数据收集
- 系统能力调用
- 性能关键型计算

## 核心能力

### 1. 跨语言绑定

| 能力 | 描述 | 证据位置 |
|------|------|----------|
| 类型转换 | C 类型与 NAPI 类型双向转换 | `native_api.h:46-68` |
| 对象封装 | JS 对象与 native 数据的绑定 | `native_node_api.h:81-140` |
| 函数调用 | JS 调用 native，native 调用 JS | `native_engine.h:224-227` |

### 2. 异步编程支持

| 能力 | 描述 | 证据位置 |
|------|------|----------|
| Async Work | 基于 libuv 的异步任务队列 | `native_async_work.h:49-98` |
| Promise | 异步结果承诺机制 | `native_deferred.h:21-26` |
| Thread-safe Function | 线程安全函数跨线程调用 | `native_node_api.h` (TSFN 系列) |
| Send Event | 任务发送到 JS 线程执行 | `native_node_api.h` |

### 3. 内存管理

| 能力 | 描述 | 证据位置 |
|------|------|----------|
| Handle Scope | 局部变量生命周期管理 | `native_engine.h` |
| Reference | JS 值的引用计数管理 | `native_reference.h` |
| Finalizer | 对象销毁回调机制 | `native_node_api.h:74-77` |

### 4. ArkTS 互操作

提供低级别 ArkTS/JS 互操作接口，支持：
- 值创建和转换（`ARKTS_CreateObject` 等）
- 属性操作（`ARKTS_GetProperty` 等）
- 函数调用（`ARKTS_Call` 等）
- Promise 能力（`ARKTS_CreatePromiseCapability` 等）

证据位置：`ark_interop_napi.h`

## 运行环境

### 硬件支持

| CPU 架构 | 支持状态 | 编译宏 |
|----------|----------|--------|
| x86_64 | 支持 | `NAPI_TARGET_AMD64`, `NAPI_TARGET_64` |
| x86 | 支持 | `NAPI_TARGET_X86`, `NAPI_TARGET_32` |
| arm64 | 支持 | `NAPI_TARGET_ARM64`, `NAPI_TARGET_64` |
| arm | 支持 | `NAPI_TARGET_ARM32`, `NAPI_TARGET_32` |

### 系统支持

| 系统 | 支持状态 | 编译宏 |
|------|----------|--------|
| OHOS 标准系统 | 支持 | `OHOS_PLATFORM`, `OHOS_STANDARD_PLATFORM` |
| OHOS 模拟器 | 支持 | `SIMULATOR` |
| Linux | 支持 | `LINUX_PLATFORM` |
| Windows (MinGW) | 支持 | `WINDOWS_PLATFORM` |
| macOS | 支持 | `MAC_PLATFORM` |
| Android (ArkUI-X) | 支持 | `ANDROID_PLATFORM` |
| iOS (ArkUI-X) | 支持 | `IOS_PLATFORM` |

### 运行时依赖

| 依赖组件 | 用途 |
|----------|------|
| ets_runtime | Ark 运行时 |
| libuv | 异步 I/O 和线程池 |
| icu | 国际化 |
| hilog | 日志 |
| hitrace | 性能追踪 |
| ace_engine | UI 框架集成 |

## 版本信息

| 属性 | 值 |
|------|------|
| N-API 版本 | 8 |
| 组件版本 | 3.1 |
| ROM 占用 | ~5120KB |
| RAM 占用 | ~10240KB |
| 许可证 | Apache-2.0 |

## 相关仓库

- [arkui_ace_engine](https://gitee.com/openharmony/arkui_ace_engine) - ArkUI 引擎
- [arkui_ace_engine_lite](https://gitee.com/openharmony/arkui_ace_engine_lite) - 轻量级引擎
- [ets_runtime](https://gitee.com/openharmony/arkcompiler/ets_runtime) - ETS 运行时
