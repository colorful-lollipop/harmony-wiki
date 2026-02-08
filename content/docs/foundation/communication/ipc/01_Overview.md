# 项目概述

## 1.1 项目定位

**OpenHarmony IPC 组件** (`foundation/communication/ipc`) 是 OpenHarmony 操作系统中实现**进程间通信（IPC）**和**远程过程调用（RPC）**的核心框架。

### 核心定位

| 维度 | 描述 |
|------|------|
| **IPC** | 使用 Binder 驱动的设备内跨进程通信 |
| **RPC** | 使用软总线（SoftBus）驱动的跨设备跨进程通信 |
| **通信模型** | 客户端-服务器（Client-Server）模式 |
| **SA 管理** | 支持系统能力（System Ability）注册与发现 |

### 关键特征

- **数据限制**: 单次 IPC 传输最大约 1MB，超大数据请使用匿名共享内存（Ashmem）
- **跨设备限制**: 不支持将跨设备的 Proxy 对象传递回原设备
- **多语言支持**: C/C++、JavaScript/ArkTS、Rust、Cangjie

### 依赖关系

```
IPC/RPC 组件
├── 依赖
│   ├── samgr（系统能力管理）
│   ├── c_utils（公共基础库）
│   ├── dsoftbus（分布式软总线）
│   ├── hilog（日志）
│   ├── hitrace（追踪）
│   └── access_token（权限）
└── 被依赖
    └── 几乎所有需要跨进程通信的系统服务
```

## 1.2 核心能力

### 能力矩阵

| 能力 | 描述 | 支持层级 |
|------|------|----------|
| **Binder IPC** | 设备内跨进程通信 | 标准/小型/轻量 |
| **DBinder** | 跨设备动态 Binder 代理 | 标准 |
| **匿名共享内存** | 大数据量传输优化 | 标准/小型 |
| **SA 注册与发现** | 系统能力生命周期管理 | 标准 |
| **死亡通知** | 远程对象存活状态监控 | 标准 |
| **调用身份追踪** | PID/UID/Token 追踪 | 标准 |

### 功能对比

| 功能 | IPC (设备内) | RPC (跨设备) |
|------|-------------|--------------|
| **传输协议** | Binder 驱动 | SoftBus 驱动 |
| **数据量限制** | ~1MB | 视网络而定 |
| **同步/异步** | 都支持 | 都支持 |
| **Proxy 传递** | 支持 | 不支持 |

## 1.3 运行环境

### 系统要求

| 要求 | 说明 |
|------|------|
| **OpenHarmony 版本** | 标准系统（standard）、小型系统（small）、轻量系统（mini） |
| **内核** | Linux Kernel / LiteOS-A / LiteOS-M |
| **运行时** | ArkTS/JS 运行时、Native runtime |

### Feature Flags

| 开关 | 默认值 | 说明 |
|------|--------|------|
| `ipc_feature_rpc_enabled` | false | 启用 RPC 功能 |
| `ipc_feature_test_enabled` | false | 启用测试功能 |
| `ipc_feature_trace_enabled` | false | 启用追踪功能 |
| `ipc_feature_freeze_enabled` | false | 启用进程冻结 |
| `ipc_feature_memory_usage_enabled` | false | 启用内存统计 |

**配置位置**: `config.gni:20-29`

## 1.4 关键概念

### 术语表

| 术语 | 英文 | 定义 |
|------|------|------|
| **Proxy** | Service Requester | 服务请求方，获取服务端代理进行通信 |
| **Stub** | Service Provider | 服务提供方，处理来自 Proxy 的请求 |
| **SA** | System Ability | 系统能力，运行在后台的系统服务 |
| **SAMgr** | System Ability Manager | 系统能力管理器，管理 SA 的注册与发现 |
| **Binder** | Binder Driver | Linux 内核驱动，实现进程间通信 |
| **DBinder** | Distributed Binder | 分布式 Binder，支持跨设备 IPC |

### 接口继承体系

```
IRemoteObject (基类)
├── IRemoteProxy<T> (模板，客户端代理)
├── IRemoteStub<T> (模板，服务端存根)
└── IPCObjectProxy / IPCObjectStub (具体实现)
```

### 消息流程

```
1. Client 获取 SA 的 Proxy
   └─ SAMgr.GetSystemAbility(said) → IRemoteObject Proxy

2. Client 通过 Proxy 发送请求
   └─ Proxy.SendRequest(code, data, reply, option)

3. Server Stub 接收并处理请求
   └─ Stub.OnRemoteRequest(code, data, reply, option)

4. Server 返回结果
   └─ reply.WriteXXX(...) → 返回给 Client
```

## 1.5 约束与限制

### 功能约束

| 约束 | 说明 |
|------|------|
| **数据量** | 单次 IPC 传输最大约 1MB |
| **跨设备 Proxy** | 不支持跨设备传递 Proxy 对象 |
| **同步调用** | 默认同步（TF_SYNC），可配置异步 |
| **文件描述符** | 需要 TF_ACCEPT_FDS 标志 |

### 兼容性约束

| 约束 | 说明 |
|------|------|
| **系统版本** | 需 OpenHarmony 3.0+ |
| **权限要求** | 敏感操作需要相应权限 |
| **ABI 兼容** | C API 保证 ABI 稳定性 |

---

*证据来源*:
- `README_zh.md:16-43` - 项目简介与约束
- `bundle.json:16-34` - 组件配置与系统适配
- `config.gni:20-29` - Feature Flags 配置
