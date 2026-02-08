# 目录结构与模块职责

## 目的

本文档描述 `communication_cangjie_wrapper` 的目录结构、各文件职责和模块边界。

## 适用范围

- 新加入开发者快速熟悉代码
- 代码维护时定位文件

## 顶层目录结构

```
foundation/communication/communication_cangjie_wrapper
├── figures/                    # 架构图（非代码）
├── kit/                        # Kit 层接口
│   └── IPCKit/
├── mock/                       # 平台 Mock 实现
├── ohos/                       # 核心实现
│   └── rpc/
├── test/                       # 测试代码（本文档忽略）
├── wiki/                       # 本文档目录
├── BUILD.gn                    # 根构建配置
├── bundle.json                 # 组件配置
├── LICENSE                     # Apache 2.0
├── README.md                   # 项目说明
└── README_zh.md                # 中文项目说明
```

## 详细目录结构（不含测试）

```
foundation/communication/communication_cangjie_wrapper/
├── figures/
│   └── communication_cangjie_wrapper_architecture_en.png
│
├── kit/
│   └── IPCKit/
│       ├── BUILD.gn            # Kit 层构建配置
│       └── index.cj            # Kit 层统一导出
│
├── mock/
│   └── ohos.rpc.cj             # Windows/Mac 模拟实现
│
├── ohos/
│   └── rpc/
│       ├── BUILD.gn            # 核心模块构建配置
│       ├── ashmem.cj           # 匿名共享内存实现
│       ├── cj_rpc_ffi.cj       # FFI 外部函数声明
│       ├── cj_rpc_utils.cj     # 错误码与工具函数
│       ├── iremote_object.cj   # IRemoteObject 接口
│       ├── message_sequence.cj # MessageSequence 实现
│       ├── parcelable.cj       # Parcelable 接口
│       ├── remote_object.cj    # RemoteObject 实现
│       ├── remote_proxy.cj     # RemoteProxy 实现
│       └── request_result.cj   # 数组类型定义
│
├── wiki/                       # 工程 Wiki（本文档）
│   ├── README.md
│   ├── SUMMARY.md
│   ├── _work/
│   └── ...
│
├── BUILD.gn                    # 根 BUILD.gn
├── bundle.json                 # 组件元数据
├── LICENSE
├── README.md
└── README_zh.md
```

## 模块职责详解

### 1. kit/IPCKit/ - Kit 接口层

**职责**: 提供开发者统一入口

**文件**: `kit/IPCKit/index.cj`

```cangjie
package kit.IPCKit
public import ohos.rpc.*  // 统一导出所有 rpc 接口
```

**构建目标**: `kit.IPCKit` (ohos_cangjie_shared_library)
- 依赖: `ohos/rpc:ohos.rpc`
- 产物: SDK Kit 库

**适用场景**: 应用开发者通过 `import kit.IPCKit` 使用 IPC 功能

---

### 2. ohos/rpc/ - 核心实现层

#### 2.1 message_sequence.cj

**职责**: MessageSequence 类完整实现

**关键符号**:
- `class MessageSequence` (行 47)
- 60+ 个 public 方法
- 支持类型: 基本类型、数组、String、FD、Ashmem、Parcelable

**核心方法分类**:
```
生命周期:
  - create(): MessageSequence$   (行 66)
  - reclaim()                    (行 81)

容量管理:
  - getSize(), getCapacity()     (行 132, 147)
  - setSize(), setCapacity()     (行 166, 185)

位置管理:
  - getReadPosition(), getWritePosition()  (行 232, 247)
  - rewindRead(), rewindWrite()            (行 284, 266)

异常处理:
  - writeNoException()           (行 301)
  - readException()              (行 319)

基本类型读写 (write/read):
  - Byte, Short, Int, Long
  - Float, Double
  - Boolean, Char
  - String

数组类型读写:
  - ByteArray, ShortArray, IntArray, LongArray
  - FloatArray, DoubleArray
  - BooleanArray, CharArray, StringArray
  - UInt8Array, UInt16Array, UInt32Array, UInt64Array

特殊类型:
  - writeFileDescriptor(), readFileDescriptor()   (行 1062, 1078)
  - writeAshmem(), readAshmem()                   (行 1095, 1111)
  - writeParcelable(), readParcelable()           (行 1319, 1342)
  - writeRawDataBuffer(), readRawDataBuffer()     (行 1413, 1437)
```

**依赖文件**:
- `cj_rpc_ffi.cj` - FFI 调用
- `cj_rpc_utils.cj` - 错误处理
- `request_result.cj` - 数组类型

---

#### 2.2 ashmem.cj

**职责**: Ashmem（匿名共享内存）类实现

**关键符号**:
- `class Ashmem` (行 34)
- 保护级别常量 (行 46-77)

**核心方法**:
```
常量定义:
  - PROT_EXEC: UInt32 = 4   // 可执行
  - PROT_NONE: UInt32 = 0   // 不可访问
  - PROT_READ: UInt32 = 1   // 可读
  - PROT_WRITE: UInt32 = 2  // 可写

创建:
  - create(name: String, size: Int32): Ashmem$     (行 89)
  - create(ashmem: Ashmem): Ashmem$                (行 113)

生命周期:
  - closeAshmem()           (行 127)
  - unmapAshmem()           (行 138)

属性:
  - getAshmemSize(): Int32  (行 150)

内存映射:
  - mapTypedAshmem(mapType: UInt32)   (行 168)
  - mapReadWriteAshmem()              (行 183)
  - mapReadonlyAshmem()               (行 198)

保护设置:
  - setProtectionType(protectionType: UInt32)  (行 214)

数据操作:
  - writeDataToAshmem(buf, size, offset)       (行 233)
  - readDataFromAshmem(size, offset)           (行 259)
```

**依赖文件**:
- `cj_rpc_ffi.cj`
- `cj_rpc_utils.cj`

---

#### 2.3 parcelable.cj

**职责**: Parcelable 序列化接口定义

**关键符号**:
- `interface Parcelable` (行 31)

**接口方法**:
```cangjie
func marshalling(dataOut: MessageSequence): Bool   // 序列化
func unmarshalling(dataIn: MessageSequence): Bool  // 反序列化
```

**使用场景**: 自定义对象需要跨进程传输时实现此接口

**依赖**: 无（纯接口定义）

---

#### 2.4 remote_object.cj

**职责**: RemoteObject（服务端 Stub）实现

**关键符号**:
- `class RemoteObjectLite` (行 22) - 基础封装
- `class RemoteObject` (行 32) - 公开类
- `class RemoteObjectImpl` (行 47) - 实现占位

**说明**: 当前版本为框架预留，功能待完善（README 中标注暂不支持远程对象通信）

---

#### 2.5 remote_proxy.cj

**职责**: RemoteProxy（客户端代理）实现

**关键符号**:
- `class RemoteProxy` (行 22)

**说明**: 当前版本为框架预留，功能待完善

---

#### 2.6 iremote_object.cj

**职责**: IRemoteObject 接口和工厂函数

**关键符号**:
- `interface IRemoteObject` (行 22)
- `func createIRemoteObject(objectId: Int64): IRemoteObject` (行 26)

**类型判断**:
```cangjie
const REMOTE_OBJCET_TYPE: Int32 = 0   // RemoteObject
const REMOTE_PROXY_TYPE: Int32 = 1    // RemoteProxy
```

---

#### 2.7 cj_rpc_ffi.cj

**职责**: 所有 FFI 外部函数声明

**关键符号**: 70+ 个 `foreign func` 声明

**分类**:
```
MessageSequence FFI (行 23-177):
  - 创建/销毁
  - Token 读写
  - 容量/位置管理
  - 基本类型读写
  - 数组类型读写
  - 原始数据读写
  - 文件描述符操作
  - 共享内存操作
  - 远程对象操作
  - 异常处理

Ashmem FFI (行 179-200):
  - 创建/关闭
  - 内存映射/解映射
  - 属性获取
  - 保护设置
  - 数据读写

RemoteObject/Proxy FFI (行 202-234):
  - 构造/描述符
  - 调用信息 (PID/UID/TokenID)
  - 死亡通知
  - 设备信息
  - IPC Skeleton 接口

工具函数 (行 236):
  - memcpy_s: 安全内存拷贝
```

**依赖**: `ohos.ffi.RetDataI64`

---

#### 2.8 cj_rpc_utils.cj

**职责**: 错误码定义、类型码定义、错误处理工具

**关键符号**:

**ErrorCode 枚举** (行 23-57):
```cangjie
enum ErrorCode {
    CheckParamError              // 401
    | OsMmapError                // 1900001
    | OsIoctlError               // 1900002
    | WriteToAshmemError         // 1900003
    | ReadFromAshmemError        // 1900004
    | OnlyProxyObjectPermittedError    // 1900005
    | OnlyRemoteObjectPermittedError   // 1900006
    | CommunicationError         // 1900007
    | ProxyOrRemoteObjectInvalidError  // 1900008
    | WriteDataToMessageSequenceError  // 1900009
    | ReadDataFromMessageSequenceError // 1900010
    | ParcelMemoryAllocError     // 1900011
    | OsDupError                 // 1900013
}
```

**TypeCode 枚举** (行 59-89):
```cangjie
enum TypeCode {
    Int8Array | Uint8Array | Int16Array | Uint16Array
    | Int32Array | Uint32Array | Float32Array | Float64Array
    | BigInt64Array | BigUint64Array
}
```

**ERR_CODE_MAP** (行 91-107): 错误码到错误信息的映射表

**checkAndThrow()** (行 109-114): 错误检查与抛出工具函数

---

#### 2.9 request_result.cj

**职责**: C 数组类型定义和转换工具

**关键符号** (全部为 `@C struct`):

| 结构体 | 用途 | 对应 Cangjie 类型 |
|--------|------|------------------|
| `ByteArray` | Int8 数组 | `Array<Int8>` |
| `ShortArray` | Int16 数组 | `Array<Int16>` |
| `CIntArray` | Int32 数组 | `Array<Int32>` |
| `LongArray` | Int64 数组 | `Array<Int64>` |
| `FloatArray` | Float32 数组 | `Array<Float32>` |
| `DoubleArray` | Float64 数组 | `Array<Float64>` |
| `CharArray` | UInt8 数组 | `Array<UInt8>` |
| `UInt16Array` | UInt16 数组 | `Array<UInt16>` |
| `UInt32Array` | UInt32 数组 | `Array<UInt32>` |
| `UInt64Array` | UInt64 数组 | `Array<UInt64>` |
| `CStringArray` | 字符串数组 | `Array<String>` |
| `RemoteObjectArray` | 远程对象数组 | `Array<IRemoteObject>` |

**通用方法**:
- `init(arr: Array<T>)` - 从 Cangjie 数组创建 C 数组
- `toArray(): Array<T>` - 转换回 Cangjie 数组
- `free(): Unit` - 释放 C 数组内存

---

### 3. mock/ - 模拟实现

#### 3.1 ohos.rpc.cj

**职责**: Windows/Mac 平台的空实现

**说明**: 在非 Linux 平台编译时使用，提供相同的 API 签名但无实际功能

**使用场景**: 开发调试阶段在非目标平台编译通过

---

### 4. 构建配置

#### 4.1 根 BUILD.gn

**职责**: 定义 SDK 库复制目标

**目标**: `copy_sdk_communication_cangjie_libs`
- 输入: ohos.rpc + kit.IPCKit
- 输出: SDK 库文件

#### 4.2 ohos/rpc/BUILD.gn

**职责**: 核心模块构建配置

**目标**: `ohos.rpc` (ohos_cangjie_shared_library)

**条件编译**:
```gn
if (is_mingw || is_mac) {
    sources = [ "../../mock/ohos.rpc.cj" ]
} else {
    sources = [ "ashmem.cj", "cj_rpc_ffi.cj", ... ]
}
```

#### 4.3 kit/IPCKit/BUILD.gn

**职责**: Kit 层构建配置

**目标**: `kit.IPCKit` (ohos_cangjie_shared_library)
- sources: `index.cj`
- cj_deps: `ohos/rpc:ohos.rpc`

---

## 模块边界

### 层级边界

```
┌─────────────────────────────────────────┐
│  Kit Layer (kit/IPCKit)                 │  应用开发者入口
│  - 统一导出                              │
├─────────────────────────────────────────┤
│  API Layer (ohos/rpc)                   │  核心 API 实现
│  - MessageSequence                       │
│  - Ashmem                                │
│  - Parcelable                            │
│  - RemoteObject/Proxy                    │
├─────────────────────────────────────────┤
│  FFI Layer (cj_rpc_ffi.cj)              │  外部接口声明
│  - foreign func 声明                     │
├─────────────────────────────────────────┤
│  Native Layer (ipc:cj_ipc_ffi)          │  外部依赖
│  - C/C++ 实现                            │
└─────────────────────────────────────────┘
```

### 依赖方向

```
kit.IPCKit ──depends──> ohos.rpc
                           │
                           ├──uses──> cj_rpc_ffi
                           │
                           ├──uses──> cj_rpc_utils
                           │
                           └──uses──> request_result

ohos.rpc ──ffi_call──> ipc:cj_ipc_ffi (外部组件)
```

**注意**: 依赖方向始终向下，无循环依赖

## 文件修改指南

### 新增 API 方法
1. 在 `cj_rpc_ffi.cj` 声明 foreign func
2. 在对应类文件（如 `message_sequence.cj`）实现 public 方法
3. 如需新错误码，在 `cj_rpc_utils.cj` 添加
4. 更新 [N-API 参考](03_NAPI_Reference.md)

### 修改错误码
1. 修改 `cj_rpc_utils.cj` 中 ErrorCode 枚举
2. 同步更新 ERR_CODE_MAP
3. 更新 [错误码附录](appendix/Error_Codes.md)

### 新增数组类型支持
1. 在 `request_result.cj` 添加新的 `@C struct`
2. 在 `cj_rpc_ffi.cj` 声明读写函数
3. 在 `message_sequence.cj` 添加 public 方法
4. 更新 TypeCode 枚举（如需要）

## 参考文档

- [架构说明](01_Architecture.md) - 组件关系图
- [N-API 参考](03_NAPI_Reference.md) - API 详细说明
- [内部 API](04_Internal_API.md) - 内部接口说明
