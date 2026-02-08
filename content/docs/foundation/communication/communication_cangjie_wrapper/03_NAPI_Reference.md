# 对外 API 参考（N-API 层）

## 目的

本文档提供 `communication_cangjie_wrapper` 对外暴露的完整 API 参考，包括参数说明、错误码和调用示例。

## 适用范围

- 应用开发者
- API 使用者

## API 命名空间

所有 API 位于 `ohos.rpc` 命名空间下，通过以下方式导入：

```cangjie
// 方式 1: 直接使用 rpc
import ohos.rpc.*

// 方式 2: 通过 Kit 层（推荐）
import kit.IPCKit.*
```

## 核心类概览

| 类名 | 文件路径 | 说明 |
|------|----------|------|
| `MessageSequence` | `ohos/rpc/message_sequence.cj:47` | RPC 数据序列化容器 |
| `Ashmem` | `ohos/rpc/ashmem.cj:34` | 匿名共享内存 |
| `Parcelable` | `ohos/rpc/parcelable.cj:31` | 可序列化接口 |
| `RemoteObject` | `ohos/rpc/remote_object.cj:32` | 远程对象（Stub）|
| `RemoteProxy` | `ohos/rpc/remote_proxy.cj:22` | 远程代理 |

## 通用信息

### API 级别

所有 API 均标记为：
- **Since**: API 22
- **Syscap**: `SystemCapability.Communication.IPC.Core`

### 异常处理

所有可能失败的方法都声明 `throwexception: true`，抛出 `BusinessException`：

```cangjie
try {
    let ms = MessageSequence.create()
    ms.writeInt(42)
} catch (e: BusinessException) {
    // e.code: 错误码
    // e.message: 错误信息
}
```

### 错误码速查

| 错误码 | 含义 | 常见场景 |
|--------|------|----------|
| 401 | 参数错误 | 空数组、越界等 |
| 1900001 | mmap 失败 | Ashmem 映射失败 |
| 1900002 | ioctl 失败 | Ashmem 保护设置失败 |
| 1900003 | Ashmem 写入失败 | 共享内存写入错误 |
| 1900004 | Ashmem 读取失败 | 共享内存读取错误 |
| 1900009 | MessageSequence 写入失败 | 写入操作错误 |
| 1900010 | MessageSequence 读取失败 | 读取操作错误 |
| 1900011 | 内存分配失败 | create() 失败 |
| 1900013 | dup 调用失败 | dupFileDescriptor 失败 |

完整错误码列表见 [错误码附录](appendix/Error_Codes.md)。

---

## MessageSequence API

### 类定义

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Communication.IPC.Core"
]
public class MessageSequence <: RemoteDataLite
```

### 静态方法

#### create()

创建空的 MessageSequence 对象。

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Communication.IPC.Core",
    throwexception: true
]
public static func create(): MessageSequence
```

**返回值**: 新创建的 MessageSequence 对象

**抛出异常**: 
- `1900011` - 内存分配失败

**示例**:
```cangjie
let ms = MessageSequence.create()
```

**实现位置**: `ohos/rpc/message_sequence.cj:66`

---

#### closeFileDescriptor()

关闭指定的文件描述符。

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Communication.IPC.Core"
]
public static func closeFileDescriptor(fd: Int32): Unit
```

**参数**:
- `fd`: 要关闭的文件描述符

**示例**:
```cangjie
MessageSequence.closeFileDescriptor(fd)
```

**实现位置**: `ohos/rpc/message_sequence.cj:1013`

---

#### dupFileDescriptor()

复制指定的文件描述符。

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Communication.IPC.Core",
    throwexception: true
]
public static func dupFileDescriptor(fd: Int32): Int32
```

**参数**:
- `fd`: 要复制的文件描述符

**返回值**: 复制后的文件描述符

**抛出异常**:
- `1900013` - dup 调用失败

**示例**:
```cangjie
let dupFd = MessageSequence.dupFileDescriptor(originalFd)
```

**实现位置**: `ohos/rpc/message_sequence.cj:1028`

---

### 生命周期方法

#### reclaim()

回收 MessageSequence 对象。

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Communication.IPC.Core"
])
public func reclaim(): Unit
```

**说明**: 释放对象占用的资源，对象不再可用。

**示例**:
```cangjie
ms.reclaim()
```

**实现位置**: `ohos/rpc/message_sequence.cj:81`

---

### 容量与位置管理

#### getSize()

获取 MessageSequence 中包含的数据大小（字节）。

```cangjie
public func getSize(): UInt32
```

**实现位置**: `ohos/rpc/message_sequence.cj:132`

---

#### getCapacity()

获取 MessageSequence 的存储容量（字节）。

```cangjie
public func getCapacity(): UInt32
```

**实现位置**: `ohos/rpc/message_sequence.cj:147`

---

#### setSize()

设置 MessageSequence 中包含的数据大小。

```cangjie
@!APILevel[throwexception: true]
public func setSize(size: UInt32): Unit
```

**抛出异常**:
- `1900009` - 写入失败（设置的大小大于容量）

**实现位置**: `ohos/rpc/message_sequence.cj:166`

---

#### setCapacity()

设置 MessageSequence 的存储容量。

```cangjie
@!APILevel[throwexception: true]
public func setCapacity(size: UInt32): Unit
```

**抛出异常**:
- `1900009` - 写入失败（设置的容量小于当前数据大小）
- `1900011` - 内存分配失败

**实现位置**: `ohos/rpc/message_sequence.cj:185`

---

#### getWritableBytes()

获取可写入的数据空间（字节）。

```cangjie
public func getWritableBytes(): UInt32
```

**计算**: `容量 - 当前数据大小`

**实现位置**: `ohos/rpc/message_sequence.cj:201`

---

#### getReadableBytes()

获取可读取的数据空间（字节）。

```cangjie
public func getReadableBytes(): UInt32
```

**计算**: `数据大小 - 已读取大小`

**实现位置**: `ohos/rpc/message_sequence.cj:217`

---

#### getReadPosition()

获取当前读取位置。

```cangjie
public func getReadPosition(): UInt32
```

**实现位置**: `ohos/rpc/message_sequence.cj:232`

---

#### getWritePosition()

获取当前写入位置。

```cangjie
public func getWritePosition(): UInt32
```

**实现位置**: `ohos/rpc/message_sequence.cj:247`

---

#### rewindWrite()

更改写入位置。注意：不正确的位置可能导致数据错误。

```cangjie
@!APILevel[throwexception: true]
public func rewindWrite(pos: UInt32): Unit
```

**参数**:
- `pos`: 目标写入位置

**抛出异常**:
- `1900009` - 写入失败

**实现位置**: `ohos/rpc/message_sequence.cj:266`

---

#### rewindRead()

更改读取位置。注意：不正确的位置可能导致数据错误。

```cangjie
@!APILevel[throwexception: true]
public func rewindRead(pos: UInt32): Unit
```

**参数**:
- `pos`: 目标读取位置

**抛出异常**:
- `1900010` - 读取失败

**实现位置**: `ohos/rpc/message_sequence.cj:284`

---

### 接口 Token 方法

#### writeInterfaceToken()

写入接口描述符 Token，用于验证接口版本。

```cangjie
@!APILevel[throwexception: true]
public func writeInterfaceToken(token: String): Unit
```

**参数**:
- `token`: 接口描述符字符串

**抛出异常**:
- `1900009` - 写入失败

**示例**:
```cangjie
ms.writeInterfaceToken("ohos.rpc.IMyService")
```

**实现位置**: `ohos/rpc/message_sequence.cj:95`

---

#### readInterfaceToken()

读取接口描述符 Token。

```cangjie
@!APILevel[throwexception: true]
public func readInterfaceToken(): String
```

**返回值**: 接口描述符字符串

**抛出异常**:
- `1900010` - 读取失败

**实现位置**: `ohos/rpc/message_sequence.cj:113`

---

### 异常处理方法

#### writeNoException()

写入无异常标记。服务端在处理请求后、写入回复数据前应调用此方法。

```cangjie
@!APILevel[throwexception: true]
public func writeNoException(): Unit
```

**抛出异常**:
- `1900009` - 写入失败

**示例**:
```cangjie
// 服务端
ms.writeNoException()  // 标记无异常
ms.writeInt(result)
```

**实现位置**: `ohos/rpc/message_sequence.cj:301`

---

#### readException()

读取异常信息。如果服务端抛出异常，将在此处抛出。

```cangjie
@!APILevel[throwexception: true]
public func readException(): Unit
```

**说明**: 应在读取其他数据前调用此方法检查异常。

**抛出异常**:
- `BusinessException` - 服务端抛出的异常
- `1900010` - 读取失败

**示例**:
```cangjie
// 客户端
ms.readException()  // 有异常会抛出
let result = ms.readInt()
```

**实现位置**: `ohos/rpc/message_sequence.cj:319`

---

### 基本类型写入方法

| 方法 | 参数 | 错误码 | 实现位置 |
|------|------|--------|----------|
| `writeByte(val: Int8)` | Int8 | 1900009 | 行 342 |
| `writeShort(val: Int16)` | Int16 | 1900009 | 行 358 |
| `writeInt(val: Int32)` | Int32 | 1900009 | 行 374 |
| `writeLong(val: Int64)` | Int64 | 1900009 | 行 390 |
| `writeFloat(val: Float32)` | Float32 | 1900009 | 行 406 |
| `writeDouble(val: Float64)` | Float64 | 1900009 | 行 422 |
| `writeBoolean(val: Bool)` | Bool | 1900009 | 行 438 |
| `writeChar(val: UInt8)` | UInt8 | 1900009 | 行 459 |
| `writeString(val: String)` | String | 1900009 | 行 475 |

---

### 基本类型读取方法

| 方法 | 返回值 | 错误码 | 实现位置 |
|------|--------|--------|----------|
| `readByte(): Int8` | Int8 | 1900010 | 行 681 |
| `readShort(): Int16` | Int16 | 1900010 | 行 698 |
| `readInt(): Int32` | Int32 | 1900010 | 行 715 |
| `readLong(): Int64` | Int64 | 1900010 | 行 732 |
| `readFloat(): Float32` | Float32 | 1900010 | 行 749 |
| `readDouble(): Float64` | Float64 | 1900010 | 行 766 |
| `readBoolean(): Bool` | Bool | 1900010 | 行 783 |
| `readChar(): UInt8` | UInt8 | 1900010 | 行 804 |
| `readString(): String` | String | 1900010 | 行 821 |

---

### 数组类型写入方法

| 方法 | 参数 | 错误码 | 实现位置 |
|------|------|--------|----------|
| `writeByteArray(byteArray: Array<Int8>)` | Int8 数组 | 1900009 | 行 493 |
| `writeShortArray(shortArray: Array<Int16>)` | Int16 数组 | 1900009 | 行 513 |
| `writeIntArray(intArray: Array<Int32>)` | Int32 数组 | 1900009 | 行 533 |
| `writeLongArray(longArray: Array<Int64>)` | Int64 数组 | 1900009 | 行 553 |
| `writeFloatArray(floatArray: Array<Float32>)` | Float32 数组 | 1900009 | 行 573 |
| `writeDoubleArray(doubleArray: Array<Float64>)` | Float64 数组 | 1900009 | 行 593 |
| `writeBooleanArray(booleanArray: Array<Bool>)` | Bool 数组 | 1900009 | 行 613 |
| `writeCharArray(charArray: Array<UInt8>)` | UInt8 数组 | 1900009 | 行 643 |
| `writeStringArray(stringArray: Array<String>)` | String 数组 | 1900009 | 行 663 |

**注意**: 数组类型不匹配或大小超出限制可能导致数据截断。

---

### 数组类型读取方法

| 方法 | 返回值 | 错误码 | 实现位置 |
|------|--------|--------|----------|
| `readByteArray(): Array<Int8>` | Int8 数组 | 1900010 | 行 842 |
| `readShortArray(): Array<Int16>` | Int16 数组 | 1900010 | 行 861 |
| `readIntArray(): Array<Int32>` | Int32 数组 | 1900010 | 行 880 |
| `readLongArray(): Array<Int64>` | Int64 数组 | 1900010 | 行 899 |
| `readFloatArray(): Array<Float32>` | Float32 数组 | 1900010 | 行 918 |
| `readDoubleArray(): Array<Float64>` | Float64 数组 | 1900010 | 行 937 |
| `readBooleanArray(): Array<Bool>` | Bool 数组 | 1900010 | 行 956 |
| `readCharArray(): Array<UInt8>` | UInt8 数组 | 1900010 | 行 975 |
| `readStringArray(): Array<String>` | String 数组 | 1900010 | 行 994 |

---

### 无符号数组写入方法

| 方法 | 参数 | 错误码 | 实现位置 |
|------|------|--------|----------|
| `writeUInt8Array(buf: Array<UInt8>)` | UInt8 数组 | 1900009 | 行 1143 |
| `writeUInt16Array(buf: Array<UInt16>)` | UInt16 数组 | 1900009 | 行 1168 |
| `writeUInt32Array(buf: Array<UInt32>)` | UInt32 数组 | 1900009 | 行 1193 |
| `writeUInt64Array(buf: Array<UInt64>)` | UInt64 数组 | 1900009 | 行 1218 |

**参数校验**: 所有无符号数组方法都检查 `buf.size == 0`，空数组会抛出错误码 1900009。

---

### 无符号数组读取方法

| 方法 | 返回值 | 错误码 | 实现位置 |
|------|--------|--------|----------|
| `readUInt8Array(): Array<UInt8>` | UInt8 数组 | 1900010 | 行 1243 |
| `readUInt16Array(): Array<UInt16>` | UInt16 数组 | 1900010 | 行 1262 |
| `readUInt32Array(): Array<UInt32>` | UInt32 数组 | 1900010 | 行 1281 |
| `readUInt64Array(): Array<UInt64>` | UInt64 数组 | 1900010 | 行 1300 |

---

### Parcelable 方法

#### writeParcelable()

写入实现了 Parcelable 接口的对象。

```cangjie
@!APILevel[throwexception: true]
public func writeParcelable<T>(val: T): Unit where T <: Parcelable
```

**参数**:
- `val`: 实现了 Parcelable 接口的对象

**抛出异常**:
- `1900009` - 写入失败（包括 marshalling 失败）

**示例**:
```cangjie
class MyData <: Parcelable {
    public func marshalling(dataOut: MessageSequence): Bool { ... }
    public func unmarshalling(dataIn: MessageSequence): Bool { ... }
}

let data = MyData()
ms.writeParcelable(data)
```

**实现位置**: `ohos/rpc/message_sequence.cj:1319`

---

#### readParcelable()

读取 Parcelable 对象。

```cangjie
@!APILevel[throwexception: true]
public func readParcelable<T>(dataIn: T): Unit where T <: Parcelable
```

**参数**:
- `dataIn`: 用于接收数据的对象（执行 unmarshalling）

**抛出异常**:
- `1900010` - 读取失败
- `1900012` - 回调函数失败

**实现位置**: `ohos/rpc/message_sequence.cj:1342`

---

#### writeParcelableArray()

写入 Parcelable 对象数组。

```cangjie
@!APILevel[throwexception: true]
public func writeParcelableArray<T>(parcelableArray: Array<T>): Unit where T <: Parcelable
```

**实现位置**: `ohos/rpc/message_sequence.cj:1361`

---

#### readParcelableArray()

读取 Parcelable 对象数组。

```cangjie
@!APILevel[throwexception: true]
public func readParcelableArray<T>(parcelableArray: Array<T>): Unit where T <: Parcelable
```

**实现位置**: `ohos/rpc/message_sequence.cj:1389`

---

### 文件描述符方法

#### writeFileDescriptor()

写入文件描述符。

```cangjie
@!APILevel[throwexception: true]
public func writeFileDescriptor(fd: Int32): Unit
```

**实现位置**: `ohos/rpc/message_sequence.cj:1062`

---

#### readFileDescriptor()

读取文件描述符。

```cangjie
@!APILevel[throwexception: true]
public func readFileDescriptor(): Int32
```

**实现位置**: `ohos/rpc/message_sequence.cj:1078`

---

#### containFileDescriptors()

检查是否包含文件描述符。

```cangjie
public func containFileDescriptors(): Bool
```

**实现位置**: `ohos/rpc/message_sequence.cj:1045`

---

### Ashmem 方法

#### writeAshmem()

写入 Ashmem 对象。

```cangjie
@!APILevel[throwexception: true]
public func writeAshmem(ashmem: Ashmem): Unit
```

**错误码**:
- `1900003` - 写入共享内存失败

**实现位置**: `ohos/rpc/message_sequence.cj:1095`

---

#### readAshmem()

读取 Ashmem 对象。

```cangjie
@!APILevel[throwexception: true]
public func readAshmem(): Ashmem
```

**错误码**:
- `1900004` - 从共享内存读取失败

**实现位置**: `ohos/rpc/message_sequence.cj:1111`

---

#### getRawDataCapacity()

获取可发送的原始数据最大容量。

```cangjie
public func getRawDataCapacity(): UInt32
```

**返回值**: 128 MB（固定值）

**实现位置**: `ohos/rpc/message_sequence.cj:1126`

---

### 原始数据方法

#### writeRawDataBuffer()

写入原始数据缓冲区。

```cangjie
@!APILevel[throwexception: true]
public func writeRawDataBuffer(rawData: Array<Byte>, size: Int64): Unit
```

**参数**:
- `rawData`: 原始数据数组
- `size`: 数据大小（必须 > 0 且 <= rawData.size）

**参数校验**:
- `size <= 0` → 抛出 401
- `size > rawData.size` → 抛出 401

**实现位置**: `ohos/rpc/message_sequence.cj:1413`

---

#### readRawDataBuffer()

读取原始数据缓冲区。

```cangjie
@!APILevel[throwexception: true]
public func readRawDataBuffer(size: Int64): Array<Byte>
```

**参数**:
- `size`: 要读取的数据大小（必须 > 0）

**参数校验**:
- `size <= 0` → 抛出 401

**实现位置**: `ohos/rpc/message_sequence.cj:1437`

---

## Ashmem API

### 类定义

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Communication.IPC.Core"
])
public class Ashmem <: RemoteDataLite
```

### 保护级别常量

| 常量 | 值 | 说明 |
|------|-----|------|
| `Ashmem.PROT_EXEC` | 4 | 可执行 |
| `Ashmem.PROT_NONE` | 0 | 不可访问 |
| `Ashmem.PROT_READ` | 1 | 可读 |
| `Ashmem.PROT_WRITE` | 2 | 可写 |

**定义位置**: `ohos/rpc/ashmem.cj:46-77`

---

### 静态创建方法

#### create(name, size)

创建指定名称和大小的 Ashmem 对象。

```cangjie
public static func create(name: String, size: Int32): Ashmem
```

**参数**:
- `name`: Ashmem 对象名称
- `size`: 大小（字节）

**参数校验**:
- `name.isEmpty()` → 抛出 401

**返回值**: 新创建的 Ashmem 对象

**实现位置**: `ohos/rpc/ashmem.cj:89`

---

#### create(ashmem)

通过复制现有 Ashmem 对象的文件描述符创建新对象。两个对象指向同一共享内存区域。

```cangjie
public static func create(ashmem: Ashmem): Ashmem
```

**实现位置**: `ohos/rpc/ashmem.cj:113`

---

### 生命周期方法

#### closeAshmem()

关闭 Ashmem 对象。

```cangjie
public func closeAshmem(): Unit
```

**实现位置**: `ohos/rpc/ashmem.cj:127`

---

#### unmapAshmem()

取消 Ashmem 对象的内存映射。

```cangjie
public func unmapAshmem(): Unit
```

**实现位置**: `ohos/rpc/ashmem.cj:138`

---

### 属性方法

#### getAshmemSize()

获取 Ashmem 内存大小。

```cangjie
public func getAshmemSize(): Int32
```

**实现位置**: `ohos/rpc/ashmem.cj:150`

---

### 内存映射方法

#### mapTypedAshmem()

按指定类型映射共享文件到虚拟地址空间。

```cangjie
@!APILevel[throwexception: true]
public func mapTypedAshmem(mapType: UInt32): Unit
```

**参数**:
- `mapType`: 保护级别（PROT_EXEC/PROT_NONE/PROT_READ/PROT_WRITE 的组合）

**错误码**:
- `1900001` - mmap 调用失败

**实现位置**: `ohos/rpc/ashmem.cj:168`

---

#### mapReadWriteAshmem()

映射为可读写的虚拟地址空间。

```cangjie
@!APILevel[throwexception: true]
public func mapReadWriteAshmem(): Unit
```

**错误码**:
- `1900001` - mmap 调用失败

**实现位置**: `ohos/rpc/ashmem.cj:183`

---

#### mapReadonlyAshmem()

映射为只读的虚拟地址空间。

```cangjie
@!APILevel[throwexception: true]
public func mapReadonlyAshmem(): Unit
```

**错误码**:
- `1900001` - mmap 调用失败

**实现位置**: `ohos/rpc/ashmem.cj:198`

---

### 保护设置方法

#### setProtectionType()

设置共享文件映射的内存区域保护级别。

```cangjie
@!APILevel[throwexception: true]
public func setProtectionType(protectionType: UInt32): Unit
```

**参数**:
- `protectionType`: 保护类型（PROT_* 常量组合）

**错误码**:
- `1900002` - ioctl 调用失败

**实现位置**: `ohos/rpc/ashmem.cj:214`

---

### 数据操作方法

#### writeDataToAshmem()

向共享内存写入数据。

```cangjie
@!APILevel[throwexception: true]
public func writeDataToAshmem(buf: Array<Byte>, size: Int64, offset: Int64): Unit
```

**参数**:
- `buf`: 要写入的数据
- `size`: 写入大小
- `offset`: 内存区域中的起始位置

**参数校验**:
- `buf.size == 0` → 抛出 1900003

**错误码**:
- `1900003` - 写入 Ashmem 失败

**实现位置**: `ohos/rpc/ashmem.cj:233`

---

#### readDataFromAshmem()

从共享内存读取数据。

```cangjie
@!APILevel[throwexception: true]
public func readDataFromAshmem(size: Int64, offset: Int64): Array<Byte>
```

**参数**:
- `size`: 读取大小
- `offset`: 内存区域中的起始位置

**错误码**:
- `1900004` - 从 Ashmem 读取失败

**实现位置**: `ohos/rpc/ashmem.cj:259`

---

## Parcelable 接口

### 接口定义

```cangjie
@!APILevel[
    since: "22",
    syscap: "SystemCapability.Communication.IPC.Core"
])
public interface Parcelable {
    func marshalling(dataOut: MessageSequence): Bool
    func unmarshalling(dataIn: MessageSequence): Bool
}
```

**定义位置**: `ohos/rpc/parcelable.cj:31-55`

---

### marshalling()

将对象序列化到 MessageSequence。

```cangjie
func marshalling(dataOut: MessageSequence): Bool
```

**参数**:
- `dataOut`: 目标 MessageSequence

**返回值**: 序列化成功返回 true

**示例**:
```cangjie
public func marshalling(dataOut: MessageSequence): Bool {
    dataOut.writeInt(this.id)
    dataOut.writeString(this.name)
    return true
}
```

---

### unmarshalling()

从 MessageSequence 反序列化对象。

```cangjie
func unmarshalling(dataIn: MessageSequence): Bool
```

**参数**:
- `dataIn`: 源 MessageSequence

**返回值**: 反序列化成功返回 true

**示例**:
```cangjie
public func unmarshalling(dataIn: MessageSequence): Bool {
    this.id = dataIn.readInt()
    this.name = dataIn.readString()
    return true
}
```

---

## 完整调用链示例

### 基本 IPC 调用

```cangjie
import ohos.rpc.*

// 创建 MessageSequence
let ms = MessageSequence.create()

// 写入数据
try {
    ms.writeInterfaceToken("IMyService")
    ms.writeInt(42)
    ms.writeString("Hello")
    ms.writeNoException()
} catch (e: BusinessException) {
    // 处理写入错误
}

// ... 通过底层 IPC 传递到服务端 ...

// 服务端读取
try {
    let token = ms.readInterfaceToken()
    let num = ms.readInt()
    let str = ms.readString()
    ms.readException()
} catch (e: BusinessException) {
    // 处理读取错误或业务异常
}

// 回收资源
ms.reclaim()
```

### Ashmem 大数据传输

```cangjie
import ohos.rpc.*

// 创建 1MB 共享内存
let ashmem = Ashmem.create("mydata", 1024 * 1024)

// 映射内存
try {
    ashmem.mapReadWriteAshmem()
} catch (e: BusinessException) {
    // 处理映射失败
}

// 写入数据
let data = Array<Byte>(1024, repeat: 0)
try {
    ashmem.writeDataToAshmem(data, Int64(data.size), 0)
} catch (e: BusinessException) {
    // 处理写入错误
}

// 通过 MessageSequence 传递 Ashmem
let ms = MessageSequence.create()
try {
    ms.writeAshmem(ashmem)
} catch (e: BusinessException) {
    // 处理错误
}

// 清理
ashmem.unmapAshmem()
ashmem.closeAshmem()
ms.reclaim()
```

### 自定义 Parcelable 对象

```cangjie
import ohos.rpc.*

class UserData <: Parcelable {
    public var id: Int32 = 0
    public var name: String = ""
    
    public func marshalling(dataOut: MessageSequence): Bool {
        try {
            dataOut.writeInt(id)
            dataOut.writeString(name)
            return true
        } catch (e: BusinessException) {
            return false
        }
    }
    
    public func unmarshalling(dataIn: MessageSequence): Bool {
        try {
            id = dataIn.readInt()
            name = dataIn.readString()
            return true
        } catch (e: BusinessException) {
            return false
        }
    }
}

// 使用
let user = UserData()
user.id = 100
user.name = "Alice"

let ms = MessageSequence.create()
ms.writeParcelable(user)

let received = UserData()
ms.readParcelable(received)
// received.id == 100, received.name == "Alice"
```

---

## 参考文档

- [目录结构](02_Directory_Structure.md) - API 实现文件位置
- [错误码附录](appendix/Error_Codes.md) - 完整错误码列表
- [安全风险评审](06_Security_Analysis.md) - API 安全分析
