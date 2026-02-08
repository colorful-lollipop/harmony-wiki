# 内部 API

## 概述

本文档描述 Cangjie SDK 的内部 API 结构，包括模块依赖关系、稳定性和实现细节。

> **注意**: 内部 API 可能在版本间发生变化，不建议在应用代码中直接使用。

## 模块依赖关系

### Kit 模块依赖图

```
kit.AbilityKit
├── ohos.app.ability
├── ohos.app.ability.ui_ability
├── ohos.bundle.bundle_manager
└── ohos.ability_access_ctrl

kit.ArkUI
├── ohos.arkui.component
├── ohos.arkui.state_management
├── ohos.window
└── ohos.display

kit.NetworkKit
├── ohos.net.http
├── ohos.net.connection
└── ohos.net.socket

kit.IPCKit
├── ohos.rpc
└── ohos.ipc
```

### 依赖方向

```
应用层 (Application)
    │
    ▼
Kit 聚合层 (kits/*.cj.d)
    │
    ▼
API 模块层 (api/{Kit}/*.cj.d)
    │
    ▼
Runtime 层 (Cangjie Runtime / ArkTS Runtime)
    │
    ▼
Native 层 (IPC / System Services)
```

## 稳定性标注

### 公开 API (Stable)

公开 API 使用 `@!APILevel` 注解标注，具有 `since` 版本号：

```cangjie
@!APILevel[
    since: "22"
]
public class PublicApi {
    public func stableMethod(): Unit
}
```

### 内部 API (Internal)

内部 API 标记为 `@Internal` 或无注解：

```cangjie
@Internal
public class InternalApi {
    public func internalMethod(): Unit
}
```

### 废弃 API (Deprecated)

废弃 API 使用 `@Deprecated` 标注：

```cangjie
@Deprecated[
    since: "22",
   替代方案: "NewApi"
]
public class DeprecatedApi { ... }
```

## Cangjie 标准库模块

### 标准库目录

```
api/Cangjie/third_party/std/
├── std.core.cj.d              # 核心类型
├── std.collection.cj.d         # 集合类型
├── std.array.cj.d             # 数组
├── std.string.cj.d            # 字符串
├── std.option.cj.d            # Option 类型
├── std.result.cj.d            # Result 类型
├── std.io.cj.d                # I/O
├── std.fs.cj.d                # 文件系统
├── std.net.cj.d               # 网络
├── std.process.cj.d           # 进程
├── std.runtime.cj.d           # 运行时
├── std.sync.cj.d              # 同步原语
├── std.crypto.digest.cj.d     # 加密摘要
├── std.crypto.cipher.cj.d     # 加密算法
├── std.math.cj.d              # 数学函数
├── std.random.cj.d            # 随机数
├── std.time.cj.d              # 时间
├── std.env.cj.d               # 环境变量
├── std.posix.cj.d             # POSIX 兼容
├── std.ffi.cj.d               # 外部函数接口
├── std.reflect.cj.d           # 反射
└── ...
```

### 核心类型

```cangjie
// Option 类型
public enum Option<T> {
    Some(T)
    None
}

// Result 类型
public enum Result<T, E> {
    Ok(T)
    Err(E)
}

// 同步原语
public class Mutex<T> {
    public func lock(): Unit
    public func unlock(): Unit
    public func with<R>(f: (T) => R): R
}
```

## 互操作 API

### N-API 类型定义

**文件**: `api/Cangjie/ohos.ark_interop.cj.d`

```cangjie
public type napi_env = CPointer<Unit>
public type napi_value = CPointer<Unit>
public type napi_callback_info = CPointer<Unit>
```

### 互操作辅助函数

**文件**: `api/Cangjie/ohos.ark_interop_helper.cj.d`

```cangjie
@!APILevel[
    since: "22"
]
public func getNapiEnv(): napi_env

@!APILevel[
    since: "22"
]
public func createNapiValue(env: napi_env, value: CPointer<Unit>): napi_value
```

## IPC 内部结构

### Ashmem (匿名共享内存)

**文件**: `api/IPCKit/ohos.rpc.cj.d`

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Communication.IPC.Core"
]
public class Ashmem {
    // 内存保护常量
    public static const PROT_READ: UInt32
    public static const PROT_WRITE: UInt32
    public static const PROT_EXEC: UInt32
    public static const PROT_NONE: UInt32

    // 创建/关闭
    public static func create(name: String, size: Int32): Ashmem
    public func closeAshmem(): Unit

    // 内存映射
    public func mapReadWriteAshmem(): Unit
    public func mapReadOnlyAshmem(): Unit
}
```

### MessageParcel (消息包裹)

```cangjie
public class MessageParcel {
    public static func create(): MessageParcel
    public func writeInt32(value: Int32): Bool
    public func readInt32(): Int32
    public func writeString(str: String): Bool
    public func readString(): String
    public func writeRemoteInterfaceToken(token: String): Bool
}
```

## 资源生命周期

### 对象生命周期

```
┌─────────────────────────────────────────────────────┐
│                    Creation                           │
│  Constructor / Factory Method / Static Method         │
└─────────────────────┬───────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│                    Usage                              │
│  Method Calls / Property Access / Iteration           │
└─────────────────────┬───────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────┐
│                    Destruction                       │
│  Close / Destroy / GC (Automatic)                    │
└─────────────────────────────────────────────────────┘
```

### 典型资源管理

```cangjie
// 文件资源
let file = File.open("test.txt", OpenFlags.RDWR)
defer {
    file.close()  // 确保关闭
}

// 网络请求
let http = HttpRequest.create()
defer {
    http.destroy()
}

// 异步资源
async {
    let camera = getCamera()
    defer {
        camera.release()
    }
    // 使用相机
}
```

## 错误传播机制

### 异常类型

```cangjie
public class BusinessException {
    public let code: Int32
    public let message: String

    public init(code: Int32, message: String)
}

// 常见错误码
// 201 - Permission denied
// 13900001 - Operation not permitted
// 13900002 - No such file or directory
// 13900012 - Permission denied
```

### 错误处理模式

```cangjie
// 方式1: 抛出异常
@!APILevel[
    since: "22",
    throwexception: true
]
public func sensitiveOperation(): ReturnType

// 方式2: 返回 Result
public func safeOperation(): Result<ReturnType, BusinessException>

// 方式3: 回调错误
public func asyncOperation(callback: (BusinessException?, ReturnType?) => Unit)
```

## 模块接口清单

### AbilityKit 模块

| 模块 | 稳定性 | 说明 |
|------|--------|------|
| `ohos.app.ability` | Stable | Ability 基类 |
| `ohos.app.ability.ui_ability` | Stable | UIAbility |
| `ohos.app.ability.want` | Stable | Want 封装 |
| `ohos.bundle.bundle_manager` | Stable | Bundle 管理 |
| `ohos.ability_access_ctrl` | Stable | 权限管理 |

### NetworkKit 模块

| 模块 | 稳定性 | 说明 |
|------|--------|------|
| `ohos.net.http` | Stable | HTTP 请求 |
| `ohos.net.connection` | Stable | 连接管理 |
| `ohos.net.socket` | Stable | Socket 通信 |

### IPCKit 模块

| 模块 | 稳定性 | 说明 |
|------|--------|------|
| `ohos.rpc` | Stable | RPC 框架 |
| `ohos.ipc` | Stable | IPC 基础 |
| `ohos.rpc.IRemoteObject` | Stable | 远程对象 |

## 相关文档

- [Kit API 参考](04_Kit_API.md)
- [系统架构](03_Architecture.md)
- [API 声明文件格式](api/AbilityKit/ohos.ability_access_ctrl.cj.d)
