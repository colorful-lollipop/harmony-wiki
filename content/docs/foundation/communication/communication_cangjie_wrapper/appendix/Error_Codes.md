# 错误码完整列表

## 目的

本文档提供 `communication_cangjie_wrapper` 组件的所有错误码参考。

## 错误码汇总

### 标准错误码（来自 ohos.business_exception）

| 错误码 | 名称 | 说明 |
|--------|------|------|
| 401 | CheckParamError | 参数错误（通用） |

### 组件特定错误码

| 错误码 | 枚举名 | 错误信息 | 触发场景 | 代码位置 |
|--------|--------|----------|----------|----------|
| 1900001 | OsMmapError | Call mmap function failed. | Ashmem 映射失败 | ashmem.cj:168-186 |
| 1900002 | OsIoctlError | Call os ioctl function failed. | Ashmem 保护设置失败 | ashmem.cj:214-217 |
| 1900003 | WriteToAshmemError | Write to ashmem failed. | Ashmem 写入失败 | ashmem.cj:233-243 |
| 1900004 | ReadFromAshmemError | Read from ashmem failed. | Ashmem 读取失败 | ashmem.cj:259-270 |
| 1900005 | OnlyProxyObjectPermittedError | Only proxy object permitted. | 操作仅限 Proxy 对象 | （预留） |
| 1900006 | OnlyRemoteObjectPermittedError | Only remote object permitted. | 操作仅限 RemoteObject | （预留） |
| 1900007 | CommunicationError | Communication failed. | IPC/RPC 通信失败 | （底层返回） |
| 1900008 | ProxyOrRemoteObjectInvalidError | Proxy or remote object is invalid. | 对象无效 | （预留） |
| 1900009 | WriteDataToMessageSequenceError | Write data to message sequence failed. | MessageSequence 写入失败 | message_sequence.cj 多处 |
| 1900010 | ReadDataFromMessageSequenceError | Read data from message sequence failed. | MessageSequence 读取失败 | message_sequence.cj 多处 |
| 1900011 | ParcelMemoryAllocError | Sequence memory alloc failed. | MessageSequence 创建失败 | message_sequence.cj:68-70 |
| 1900012 | JsCallBackError | Failed to call the JS callback function. | JS 回调失败 | （预留，未使用） |
| 1900013 | OsDupError | Call os dup function failed. | dup 调用失败 | message_sequence.cj:1028-1033 |

## 错误码定义源码

```cangjie
// ohos/rpc/cj_rpc_utils.cj:23-57

enum ErrorCode {
    CheckParamError
    | OsMmapError
    | OsIoctlError
    | WriteToAshmemError
    | ReadFromAshmemError
    | OnlyProxyObjectPermittedError
    | OnlyRemoteObjectPermittedError
    | CommunicationError
    | ProxyOrRemoteObjectInvalidError
    | WriteDataToMessageSequenceError
    | ReadDataFromMessageSequenceError
    | ParcelMemoryAllocError
    | OsDupError

    prop value: Int32 {
        get() {
            match (this) {
                case CheckParamError => 401
                case OsMmapError => 1900001
                case OsIoctlError => 1900002
                case WriteToAshmemError => 1900003
                case ReadFromAshmemError => 1900004
                case OnlyProxyObjectPermittedError => 1900005
                case OnlyRemoteObjectPermittedError => 1900006
                case CommunicationError => 1900007
                case ProxyOrRemoteObjectInvalidError => 1900008
                case WriteDataToMessageSequenceError => 1900009
                case ReadDataFromMessageSequenceError => 1900010
                case ParcelMemoryAllocError => 1900011
                case OsDupError => 1900013
            }
        }
    }
}
```

## 错误信息映射表

```cangjie
// ohos/rpc/cj_rpc_utils.cj:91-107

let ERR_CODE_MAP = HashMap<Int32, String>(
    [
        (ErrorCode.CheckParamError.value, "Parameter error."),
        (ErrorCode.OsMmapError.value, "Call mmap function failed."),
        (ErrorCode.OsIoctlError.value, "Call os ioctl function failed."),
        (ErrorCode.WriteToAshmemError.value, "Write to ashmem failed."),
        (ErrorCode.ReadFromAshmemError.value, "Read from ashmem failed."),
        (ErrorCode.OnlyProxyObjectPermittedError.value, "Only proxy object permitted."),
        (ErrorCode.OnlyRemoteObjectPermittedError.value, "Only remote object permitted."),
        (ErrorCode.CommunicationError.value, "Communication failed."),
        (ErrorCode.ProxyOrRemoteObjectInvalidError.value, "Proxy or remote object is invalid."),
        (ErrorCode.WriteDataToMessageSequenceError.value, "Write data to message sequence failed."),
        (ErrorCode.ReadDataFromMessageSequenceError.value, "Read data from message sequence failed."),
        (ErrorCode.ParcelMemoryAllocError.value, "Sequence memory alloc failed."),
        (ErrorCode.OsDupError.value, "Call os dup function failed.")
    ]
)
```

## 错误处理工具函数

```cangjie
// ohos/rpc/cj_rpc_utils.cj:109-114

func checkAndThrow(errCode: Int32): Unit {
    if (errCode != 0) {
        let msg = ERR_CODE_MAP[errCode]
        throw BusinessException(errCode, msg)
    }
}
```

## 按 API 分类的错误码

### MessageSequence API

| 方法 | 可能错误码 |
|------|-----------|
| create() | 1900011 |
| write*() | 1900009 |
| read*() | 1900010 |
| setSize(), setCapacity() | 1900009 |
| rewindWrite(), rewindRead() | 1900009, 1900010 |
| writeInterfaceToken() | 1900009 |
| readInterfaceToken() | 1900010 |
| writeNoException() | 1900009 |
| readException() | 1900010 |
| writeRawDataBuffer() | 1900009, 401 |
| readRawDataBuffer() | 1900010, 401 |
| writeParcelable() | 1900009, 1900012 |
| readParcelable() | 1900010, 1900012 |
| dupFileDescriptor() | 1900013 |
| writeAshmem() | 1900003 |
| readAshmem() | 1900004 |

### Ashmem API

| 方法 | 可能错误码 |
|------|-----------|
| create() | 401 |
| mapTypedAshmem() | 1900001 |
| mapReadWriteAshmem() | 1900001 |
| mapReadonlyAshmem() | 1900001 |
| setProtectionType() | 1900002 |
| writeDataToAshmem() | 1900003, 401 |
| readDataFromAshmem() | 1900004 |

## 错误码使用示例

```cangjie
import ohos.rpc.*
import ohos.business_exception.BusinessException

try {
    let ms = MessageSequence.create()
    ms.writeInt(42)
} catch (e: BusinessException) {
    match (e.code) {
        case 1900011 => {
            // 内存分配失败
            println("内存不足，请释放资源后重试")
        }
        case 1900009 => {
            // 写入失败
            println("数据写入失败，可能容量不足")
        }
        case _ => {
            println("未知错误: ${e.code}, ${e.message}")
        }
    }
}
```

## 参考文档

- [N-API 参考](../03_NAPI_Reference.md) - 各 API 的错误码详情
- [常见问题](../08_Troubleshooting.md) - 基于错误码的问题诊断
