# 项目概述

## 目的

本文档提供 `communication_cangjie_wrapper` 组件的高层次概览，帮助开发者快速理解项目定位、核心能力和运行环境。

## 适用范围

- 新加入的开发者
- 技术方案评审人员
- 系统集成工程师

## 项目定位

### 一句话描述

`communication_cangjie_wrapper` 是 OpenHarmony 平台为 Cangjie 语言提供的 IPC/RPC 跨进程通信能力封装库。

### 在 OpenHarmony 中的位置

```
┌─────────────────────────────────────────────────────────────┐
│                    Application Layer                        │
│              (Cangjie Apps using IPCKit)                    │
└──────────────────────┬──────────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────────┐
│              IPCKit (kit/IPCKit/index.cj)                   │
│              统一导出 ohos.rpc.* 接口                        │
└──────────────────────┬──────────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────────┐
│       communication_cangjie_wrapper (本组件)                 │
│  ┌──────────────────────────────────────────────────────┐  │
│  │  MessageSequence  │  Ashmem  │  Parcelable           │  │
│  └──────────────────────────────────────────────────────┘  │
└──────────────────────┬──────────────────────────────────────┘
                       │ FFI (Foreign Function Interface)
┌──────────────────────▼──────────────────────────────────────┐
│              ipc:cj_ipc_ffi (C/C++ 层)                       │
│              底层 IPC/RPC 实现                               │
└──────────────────────┬──────────────────────────────────────┘
                       │
┌──────────────────────▼──────────────────────────────────────┐
│              Kernel (Binder / SoftBus Driver)                │
└─────────────────────────────────────────────────────────────┘
```

## 核心能力

### 1. 跨进程通信（IPC）
- **机制**: 使用 Binder 驱动
- **范围**: 设备内跨进程通信
- **典型场景**: 后台服务提供跨进程服务调用能力

### 2. 远程过程调用（RPC）
- **机制**: 使用软总线（SoftBus）驱动
- **范围**: 跨设备跨进程通信
- **典型场景**: 多端协同提供远程接口调用和数据传输

### 3. 匿名共享内存（Ashmem）
- **用途**: 传输超过 200KB 的大数据
- **能力**: 创建、映射、读写、保护设置

### 4. 消息序列化（MessageSequence）
- **支持类型**: 基本类型、数组、字符串、文件描述符、Ashmem、Parcelable
- **默认容量**: 200KB
- **最大原始数据**: 128MB

## 运行环境

### 支持设备类型
- ✅ 标准设备（standard）
- ❌ 轻量设备（当前分布式软总线接口不支持）

### 系统要求
- OpenHarmony API Level 22+
- SystemCapability.Communication.IPC.Core

### 开发语言
- Cangjie（仓颉编程语言）
- 通过 FFI 与底层 C/C++ IPC 实现交互

## 关键概念

### MessageSequence（消息序列）

RPC/IPC 过程中的数据载体，提供：
- **写入端**: 使用 `write*` 方法将数据按特定格式写入
- **读取端**: 使用 `read*` 方法按特定格式读取数据

### Ashmem（匿名共享内存）

用于大容量数据传输：
- **创建**: `Ashmem.create(name, size)`
- **映射**: `mapReadWriteAshmem()` / `mapReadonlyAshmem()`
- **保护级别**: PROT_READ / PROT_WRITE / PROT_EXEC / PROT_NONE

### Parcelable（可序列化接口）

自定义对象的序列化契约：
```cangjie
public interface Parcelable {
    func marshalling(dataOut: MessageSequence): Bool
    func unmarshalling(dataIn: MessageSequence): Bool
}
```

### RemoteObject / RemoteProxy

- **RemoteObject**: 服务端 Stub 实现基类
- **RemoteProxy**: 客户端代理实现基类

## 约束与限制

### 数据传输限制
| 项目 | 限制 | 说明 |
|------|------|------|
| 单次 IPC 数据上限 | 200KB | 超过需使用 Ashmem |
| Ashmem 原始数据上限 | 128MB | `getRawDataCapacity()` 返回值 |

### 功能限制（对比 ArkTS 版本）
| 功能 | 支持状态 | 备注 |
|------|----------|------|
| 基本类型传输 | ✅ 支持 | Int8/16/32/64, Float32/64, Bool, UInt8, String |
| 数组传输 | ✅ 支持 | 所有基本类型数组 |
| 文件描述符 | ✅ 支持 | FD 传递 |
| Ashmem | ✅ 支持 | 共享内存 |
| Parcelable | ✅ 支持 | 自定义序列化对象 |
| **远程对象通信** | ❌ **不支持** | RemoteObject 作为参数传递 |
| **OneWay 调用** | ❌ **不支持** | 异步单向调用 |

### 平台限制
- Windows/Mac 平台使用 Mock 实现（仅接口空实现）
- 仅 Linux/Android 内核有完整功能

## 相关仓库

| 仓库 | 作用 | 依赖关系 |
|------|------|----------|
| [arkcompiler_cangjie_ark_interop](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop) | Cangjie FFI 和注解定义 | 本组件依赖 |
| [communication_ipc](https://gitcode.com/openharmony/communication_ipc) | IPC/RPC C/C++ 实现 | 本组件依赖 |
| [hiviewdfx_hiviewdfx_cangjie_wrapper](https://gitcode.com/openharmony-sig/hiviewdfx_hiviewdfx_cangjie_wrapper) | 日志接口 | 本组件依赖 |

## 关键指标

| 指标 | 数值 | 来源 |
|------|------|------|
| ROM | 300KB | bundle.json:21 |
| RAM | 228KB | bundle.json:22 |
| API 数量 | 80+ | MessageSequence (60+) + Ashmem (10+) + 其他 |
| 错误码 | 13 个 | cj_rpc_utils.cj:23-57 |

## 下一步阅读

- [架构说明](01_Architecture.md) - 深入理解组件关系
- [目录结构](02_Directory_Structure.md) - 了解代码组织
- [N-API 参考](03_NAPI_Reference.md) - 学习 API 使用
