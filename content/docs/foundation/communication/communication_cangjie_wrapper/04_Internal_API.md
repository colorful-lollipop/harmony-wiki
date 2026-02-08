# 内部 API 与架构

## 目的

本文档描述 `communication_cangjie_wrapper` 内部模块的接口、依赖关系和实现细节，适用于核心开发者深入理解代码。

## 适用范围

- 核心模块开发者
- 代码审查人员
- 维护工程师

## 模块概述

### 模块清单

| 模块 | 文件 | 职责 | 访问级别 |
|------|------|------|----------|
| FFI Bridge | `cj_rpc_ffi.cj` | 声明所有外部 C 函数 | 内部 |
| 错误处理 | `cj_rpc_utils.cj` | 错误码定义与工具函数 | 内部 |
| 数组类型 | `request_result.cj` | C 数组包装结构体 | 内部 |
| 远程对象接口 | `iremote_object.cj` | IRemoteObject 接口定义 | protected |
| RemoteObject | `remote_object.cj` | 服务端 Stub 基类 | protected |
| RemoteProxy | `remote_proxy.cj` | 客户端代理基类 | protected |

### 访问级别说明

```cangjie
// public - 对外暴露
public class MessageSequence

// protected - 仅模块内和子类可访问
protected interface IRemoteObject
protected class RemoteObject

// private - 仅类内可访问
private var lite: ?RemoteObjectLite
```

## FFI 层详解

### 文件: `cj_rpc_ffi.cj`

**职责**: 声明所有与底层 C/C++ IPC 实现交互的 FFI 函数。

**特点**:
- 无逻辑代码，纯声明
- 使用 Cangjie `foreign` 关键字
- 类型映射：Cangjie 类型 ↔ C 类型

### MessageSequence FFI 函数

#### 创建与管理

```cangjie
foreign func FfiRpcMessageSequenceImplCreate(): Int64
```

**说明**: 创建 MessageSequence 对象，返回对象 ID（Int64）。

**调用位置**: `message_sequence.cj:67`

---

#### 接口 Token

```cangjie
foreign func FfiRpcMessageSequenceImplWriteInterfaceToken(
    id: Int64, 
    tokemn: CString, 
    errCode: CPointer<Int32>
): Unit

foreign func FfiRpcMessageSequenceImplReadInterfaceToken(
    id: Int64, 
    errCode: CPointer<Int32>
): CString
```

**说明**: 读写接口描述符 Token。

**调用位置**: 
- Write: `message_sequence.cj:98`
- Read: `message_sequence.cj:115`

---

#### 容量与位置

```cangjie
foreign func FfiRpcMessageSequenceImplGetSize(id: Int64, errCode: CPointer<Int32>): UInt32
foreign func FfiRpcMessageSequenceImplGetCapacity(id: Int64, errCode: CPointer<Int32>): UInt32
foreign func FfiRpcMessageSequenceImplSetSize(id: Int64, value: UInt32, errCode: CPointer<Int32>): Unit
foreign func FfiRpcMessageSequenceImplSetCapacity(id: Int64, value: UInt32, errCode: CPointer<Int32>): Unit
foreign func FfiRpcMessageSequenceImplGetWritableBytes(id: Int64, errCode: CPointer<Int32>): UInt32
foreign func FfiRpcMessageSequenceImplGetReadableBytes(id: Int64, errCode: CPointer<Int32>): UInt32
foreign func FfiRpcMessageSequenceImplGetReadPosition(id: Int64, errCode: CPointer<Int32>): UInt32
foreign func FfiRpcMessageSequenceImplGetWritePosition(id: Int64, errCode: CPointer<Int32>): UInt32
foreign func FfiRpcMessageSequenceImplRewindWrite(id: Int64, pos: UInt32, errCode: CPointer<Int32>): Unit
foreign func FfiRpcMessageSequenceImplRewindRead(id: Int64, pos: UInt32, errCode: CPointer<Int32>): Unit
```

**调用位置**: 对应 MessageSequence 的同名方法。

---

#### 基本类型读写

```cangjie
// 写入
foreign func FfiRpcMessageSequenceImplWriteByte(id: Int64, value: Int8, errCode: CPointer<Int32>): Unit
foreign func FfiRpcMessageSequenceImplWriteShort(id: Int64, value: Int16, errCode: CPointer<Int32>): Unit
foreign func FfiRpcMessageSequenceImplWriteInt(id: Int64, value: Int32, errCode: CPointer<Int32>): Unit
foreign func FfiRpcMessageSequenceImplWriteLong(id: Int64, value: Int64, errCode: CPointer<Int32>): Unit
foreign func FfiRpcMessageSequenceImplWriteFloat(id: Int64, value: Float32, errCode: CPointer<Int32>): Unit
foreign func FfiRpcMessageSequenceImplWriteDouble(id: Int64, value: Float64, errCode: CPointer<Int32>): Unit
foreign func FfiRpcMessageSequenceImplWriteBoolean(id: Int64, value: Int8, errCode: CPointer<Int32>): Unit
foreign func FfiRpcMessageSequenceImplWriteChar(id: Int64, value: UInt8, errCode: CPointer<Int32>): Unit
foreign func FfiRpcMessageSequenceImplWriteString(id: Int64, value: CString, errCode: CPointer<Int32>): Unit

// 读取（类似模式，略）
```

**类型映射**:
| Cangjie | C | 说明 |
|---------|---|------|
| `Int8` | `int8_t` | 有符号字节 |
| `Int16` | `int16_t` | 短整型 |
| `Int32` | `int32_t` | 整型 |
| `Int64` | `int64_t` | 长整型 |
| `Float32` | `float` | 单精度浮点 |
| `Float64` | `double` | 双精度浮点 |
| `UInt8` | `uint8_t` | 无符号字节 |
| `CString` | `char*` | 以 null 结尾的字符串 |

---

#### 数组类型读写

```cangjie
foreign func FfiRpcMessageSequenceImplWriteByteArray(
    id: Int64, 
    value: ByteArray, 
    errCode: CPointer<Int32>
): Unit

// 其他数组类型类似：ShortArray, CIntArray, LongArray, 
// FloatArray, DoubleArray, CharArray, CStringArray
```

**类型说明**: `ByteArray` 等定义在 `request_result.cj`，使用 `@C` 标记与 C 结构体兼容。

---

#### 原始数据缓冲区

```cangjie
foreign func FfiRpcMessageSequenceImplWriteRawDataBuffer(
    id: Int64, 
    data: CPointer<UInt8>, 
    size: Int64,
    errCode: CPointer<Int32>
): Unit

foreign func FfiRpcMessageSequenceImplReadRawDataBuffer(
    id: Int64, 
    size: Int64, 
    errCode: CPointer<Int32>
): CPointer<UInt8>
```

---

#### 文件描述符

```cangjie
foreign func FfiRpcMessageSequenceImplWriteFileDescriptor(
    id: Int64, 
    fd: Int32, 
    errCode: CPointer<Int32>
): Unit

foreign func FfiRpcMessageSequenceImplReadFileDescriptor(
    id: Int64, 
    errCode: CPointer<Int32>
): Int32

foreign func FfiRpcMessageSequenceImplContainFileDescriptors(
    id: Int64, 
    errCode: CPointer<Int32>
): Bool

foreign func FfiRpcMessageSequenceImplCloseFileDescriptor(fd: Int32): Unit
foreign func FfiRpcMessageSequenceImplDupFileDescriptor(fd: Int32): Int32
```

---

#### 共享内存

```cangjie
foreign func FfiRpcMessageSequenceImplWriteAshmem(
    mid: Int64, 
    aid: Int64, 
    errCode: CPointer<Int32>
): Unit

foreign func FfiRpcMessageSequenceImplReadAshmem(
    id: Int64, 
    errCode: CPointer<Int32>
): Int64

foreign func FfiRpcMessageSequenceImplGetRawDataCapacity(
    id: Int64, 
    errCode: CPointer<Int32>
): UInt32
```

---

#### 远程对象

```cangjie
foreign func FfiRpcMessageSequenceImplWriteRemoteObject(
    id: Int64, 
    object: Int64, 
    errCode: CPointer<Int32>
): Unit

foreign func FfiRpcMessageSequenceImplReadRemoteObject(
    id: Int64, 
    errCode: CPointer<Int32>
): RetDataI64

foreign func FfiRpcMessageSequenceImplWriteRemoteObjectArray(
    id: Int64, 
    value: LongArray, 
    errCode: CPointer<Int32>
): Unit

foreign func FfiRpcMessageSequenceImplReadRemoteObjectArray(
    id: Int64, 
    errCode: CPointer<Int32>
): RemoteObjectArray
```

**说明**: `RetDataI64` 定义在 `ohos.ffi`，用于返回可能失败的数据。

---

#### 异常处理

```cangjie
foreign func FfiRpcMessageSequenceImplWriteNoException(
    id: Int64, 
    errCode: CPointer<Int32>
): Unit

foreign func FfiRpcMessageSequenceImplReadException(
    id: Int64, 
    errCode: CPointer<Int32>
): CString
```

---

### Ashmem FFI 函数

```cangjie
// 创建
foreign func FfiRpcAshmemImplCreate(ashmemName: CString, ashmemSize: Int32): Int64
foreign func FfiRpcAshmemImplCreateFromExisting(id: Int64, errCode: CPointer<Int32>): Int64

// 关闭与映射
foreign func FfiRpcAshmemImplCloseAshmem(id: Int64): Unit
foreign func FfiRpcAshmemImplUnmapAshmem(id: Int64): Unit

// 属性
foreign func FfiRpcAshmemImplGetAshmemSize(id: Int64, errCode: CPointer<Int32>): Int32

// 映射操作
foreign func FfiRpcAshmemImplMapTypedAshmem(
    id: Int64, 
    mapType: UInt32, 
    errCode: CPointer<Int32>
): Unit

foreign func FfiRpcAshmemImplMapReadWriteAshmem(id: Int64, errCode: CPointer<Int32>): Unit
foreign func FfiRpcAshmemImplMapReadonlyAshmem(id: Int64, errCode: CPointer<Int32>): Unit

// 保护设置
foreign func FfiRpcAshmemImplSetProtectionType(
    id: Int64, 
    protectionType: UInt32, 
    errCode: CPointer<Int32>
): Unit

// 数据读写
foreign func FfiRpcAshmemImplWriteDataToAshmem(
    id: Int64, 
    data: CPointer<UInt8>, 
    size: Int64, 
    offset: Int64,
    errCode: CPointer<Int32>
): Unit

foreign func FfiRpcAshmemImplReadDataFromAshmem(
    id: Int64, 
    size: Int64, 
    offset: Int64, 
    errCode: CPointer<Int32>
): CPointer<UInt8>
```

---

### RemoteObject FFI 函数

```cangjie
// 构造与描述符
foreign func FfiRpcRemoteObjectConstructor(stringValue: CString): Int64
foreign func FfiRpcRemoteObjectGetDescriptor(id: Int64, errCode: CPointer<Int32>): CString
foreign func FfiRpcRemoteObjectModifyLocalInterface(
    id: Int64, 
    stringValue: CString, 
    errCode: CPointer<Int32>
): Unit

// 调用信息
foreign func FfiRpcRemoteObjectGetCallingPid(): Int32
foreign func FfiRpcRemoteObjectGetCallingUid(): Int32
```

---

### RemoteProxy FFI 函数

```cangjie
// 死亡通知
foreign func FfiRpcRemoteProxyRegisterDeathRecipient(
    id: Int64, 
    funcId: Int64, 
    flag: Int32, 
    errCode: CPointer<Int32>
): Unit

foreign func FfiRpcRemoteProxyUnregisterDeathRecipient(
    id: Int64, 
    funcId: Int64, 
    flag: Int32, 
    errCode: CPointer<Int32>
): Unit

// 查询
foreign func FfiRpcRemoteProxyGetDescriptor(id: Int64, errCode: CPointer<Int32>): CString
foreign func FfiRpcRemoteProxyIsObjectDead(id: Int64): Bool
```

---

### IPC Skeleton FFI 函数

```cangjie
// 上下文
foreign func FfiRpcIPCSkeletonGetContextObject(): RetDataI64
foreign func FfiRpcIPCSkeletonFlushCmdBuffer(object: Int64): Unit

// 调用者信息
foreign func FfiRpcIPCSkeletonGetCallingPid(): Int32
foreign func FfiRpcIPCSkeletonGetCallingUid(): Int32
foreign func FfiRpcIPCSkeletonGetCallingTokenId(): UInt32

// 设备信息
foreign func FfiRpcIPCSkeletonGetCallingDeviceID(): CString
foreign func FfiRpcIPCSkeletonGetLocalDeviceID(): CString
foreign func FfiRpcIPCSkeletonIsLocalCalling(): Bool
```

---

### 工具函数

```cangjie
foreign func memcpy_s(
    dest: CPointer<UInt8>, 
    destMax: UIntNative, 
    src: CPointer<UInt8>, 
    count: UIntNative
): Int32
```

**说明**: 安全内存拷贝函数，防止缓冲区溢出。

---

## 错误处理模块

### 文件: `cj_rpc_utils.cj`

**职责**: 定义错误码、类型码，提供错误处理工具。

### ErrorCode 枚举

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

    prop value: Int32 { ... }
}
```

**错误码范围**: 1900001 - 1900013

### checkAndThrow() 函数

```cangjie
func checkAndThrow(errCode: Int32): Unit {
    if (errCode != 0) {
        let msg = ERR_CODE_MAP[errCode]
        throw BusinessException(errCode, msg)
    }
}
```

**使用模式**: 所有 FFI 调用后都调用此函数检查错误。

---

## 数组类型模块

### 文件: `request_result.cj`

**职责**: 定义与 C 兼容的数组结构体，用于 FFI 传递。

### 通用结构模式

所有数组结构体遵循相同模式：

```cangjie
@C
struct XxxArray {
    let data: CPointer<T>   // 指向 C 数组的指针
    let len: UInt32          // 数组长度

    // 从 Cangjie 数组构造
    init(arr: Array<T>) { ... }

    // 转换回 Cangjie 数组
    func toArray(): Array<T> { ... }

    // 释放 C 内存
    func free(): Unit { ... }
}
```

### CStringArray 特殊处理

字符串数组需要处理嵌套内存分配：

```cangjie
@C
struct CStringArray {
    let data: CPointer<CString㸀3e
    let len: UInt32

    init(arr: Array<String>) {
        // 每个字符串单独分配内存
        for (i in 0..arr.size) {
            data.write(i, LibC.mallocCString(arr[i]))
        }
    }

    func free(): Unit {
        // 先释放每个字符串，再释放数组
        for (i in 0..Int64(len)) {
            LibC.free(data.read(i))
        }
        LibC.free(data)
    }
}
```

**安全考虑**: 构造时若中途失败，需要回滚已分配的内存。

---

## 远程对象模块

### 类层次结构

```
RemoteDataLite (from ohos.ffi)
    ├── RemoteObjectLite (remote_object.cj:22)
    │       └── RemoteObject (remote_object.cj:32)
    ├── RemoteProxy (remote_proxy.cj:22)
    ├── MessageSequence (message_sequence.cj:47)
    └── Ashmem (ashmem.cj:34)

IRemoteObject (iremote_object.cj:22)
    ├── RemoteObject (实现)
    └── RemoteProxy (实现)
```

### RemoteObjectLite

```cangjie
class RemoteObjectLite <: RemoteDataLite {
    protected init(id: Int64) { super(id) }
    ~init() { releaseFFIData(myDataId) }
}
```

**职责**: 封装 FFI 对象 ID，自动释放资源。

### RemoteObject

```cangjie
protected open class RemoteObject <: IRemoteObject {
    private var lite: ?RemoteObjectLite = None
    private let impl_: RemoteObjectImpl = RemoteObjectImpl()

    protected init(id: Int64) {
        lite = RemoteObjectLite(id)
    }

    protected func getID(): Int64 {
        return lite?.getID() ?? 0
    }
}
```

**状态**: 框架预留，当前版本功能未完善。

### RemoteProxy

```cangjie
protected class RemoteProxy <: RemoteDataLite & IRemoteObject {
    protected init(id: Int64) { super(id) }
    ~init() { releaseFFIData(myDataId) }
}
```

**状态**: 框架预留，当前版本功能未完善。

### IRemoteObject 工厂

```cangjie
protected func createIRemoteObject(objectId: Int64): IRemoteObject {
    const REMOTE_OBJCET_TYPE: Int32 = 0
    const REMOTE_PROXY_TYPE: Int32 = 1
    let remoteType = unsafe { FfiRpcGetRemoteType(objectId) }
    let res: IRemoteObject = if (remoteType == REMOTE_OBJCET_TYPE) {
        RemoteObject(objectId)
    } else if (remoteType == REMOTE_PROXY_TYPE) {
        RemoteProxy(objectId)
    } else {
        throw BusinessException(remoteType, "error remote type")
    }
    return res
}
```

---

## 依赖关系图

### 文件依赖图

```
cj_rpc_ffi.cj ─────────┐
                       │
cj_rpc_utils.cj ───────┼──> message_sequence.cj
                       │
request_result.cj ─────┘


cj_rpc_ffi.cj ───> ashmem.cj

cj_rpc_ffi.cj ───> remote_object.cj
cj_rpc_ffi.cj ───> remote_proxy.cj

remote_object.cj ───> iremote_object.cj
remote_proxy.cj ───> iremote_object.cj

parcelable.cj ─── (无依赖)
```

### 外部依赖图

```
ohos.rpc ───depends──> cangjie_ark_interop:ohos.ffi
           ───depends──> cangjie_ark_interop:ohos.labels
           ───depends──> cangjie_ark_interop:ohos.business_exception
           ───depends──> hiviewdfx_cangjie_wrapper:ohos.hilog
           ───depends──> ipc:cj_ipc_ffi
```

---

## 稳定性说明

### 稳定接口（可依赖）

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| MessageSequence API | 稳定 | 已完整实现 |
| Ashmem API | 稳定 | 已完整实现 |
| Parcelable 接口 | 稳定 | 纯接口定义 |
| ErrorCode | 稳定 | 错误码固定 |

### 不稳定接口（预留/待完善）

| 接口 | 稳定性 | 说明 |
|------|--------|------|
| RemoteObject | 不稳定 | README 标注不支持 |
| RemoteProxy | 不稳定 | README 标注不支持 |
| IRemoteObject | 不稳定 | 框架预留 |

### 可替换点

1. **FFI 层**: 可替换为其他 IPC 实现（需保持接口兼容）
2. **Mock 层**: Windows/Mac 已实现可替换版本
3. **错误码**: 可扩展新错误码（保持向后兼容）

---

## 参考文档

- [目录结构](02_Directory_Structure.md) - 文件组织
- [N-API 参考](03_NAPI_Reference.md) - 对外 API 详情
- [GN Targets](05_GN_Targets.md) - 构建目标
