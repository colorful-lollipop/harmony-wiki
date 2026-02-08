# 常见问题与调试

## 目的

本文档提供 `communication_cangjie_wrapper` 组件的常见问题、诊断方法和调试技巧。

## 适用范围

- 应用开发者
- 系统集成人员
- 问题定位工程师

## 问题分类速查

| 问题现象 | 可能原因 | 快速检查 |
|----------|----------|----------|
| `create()` 失败 | 内存不足 | 检查内存 / 错误码 1900011 |
| 数据写入失败 | 容量不足 | 检查容量 / 错误码 1900009 |
| Ashmem 映射失败 | mmap 错误 | 检查权限 / 错误码 1900001 |
| IPC 调用失败 | 连接问题 | 检查服务状态 / 错误码 1900007 |
| FD 传递失败 | 无效 FD | 检查 FD 值 / 错误码 1900013 |

## 常见构建问题

### 问题 1: 找不到依赖库

**现象**:
```
ERROR: //foundation/communication/communication_cangjie_wrapper/ohos/rpc:ohos.rpc 
  dependency //foundation/communication/ipc:cj_ipc_ffi not found
```

**原因**: `ipc` 组件未编译或路径错误

**解决**:
```bash
# 1. 确认 ipc 组件存在
ls foundation/communication/ipc/BUILD.gn

# 2. 先编译依赖
./build.sh --product {product} --target "//foundation/communication/ipc"

# 3. 再编译本组件
./build.sh --product {product} \
    --target "//foundation/communication/communication_cangjie_wrapper/..."
```

---

### 问题 2: Windows/Mac 编译功能缺失

**现象**: 在非 Linux 平台编译后，IPC 功能无法使用

**原因**: 设计行为，Windows/Mac 使用 Mock 实现

**代码证据**: `ohos/rpc/BUILD.gn:21-35`

```gn
if (is_mingw || is_mac){
  sources = [ "../../mock/ohos.rpc.cj" ]
} else {
  sources = [ "ashmem.cj", ... ]
}
```

**解决**: 在 Linux/OpenHarmony 环境下编译和运行

---

### 问题 3: Cangjie 编译器版本不匹配

**现象**:
```
Error: Unknown annotation @!APILevel
```

**原因**: Cangjie 编译器版本过低，不支持 APILevel 注解

**解决**: 升级 Cangjie 编译器到支持 API Level 22 的版本

---

## 常见运行时问题

### 问题 4: MessageSequence.create() 失败

**现象**:
```
BusinessException: code=1900011, message="Sequence memory alloc failed."
```

**代码位置**: `ohos/rpc/message_sequence.cj:68-70`

```cangjie
let id: Int64 = unsafe { FfiRpcMessageSequenceImplCreate() }
if (id < 0) {
    throw BusinessException(1900011, "Memory allocation failed.")
}
```

**原因**:
1. 系统内存不足
2. 达到进程资源限制
3. 底层 IPC 服务异常

**诊断**:
```bash
# 检查系统内存
cat /proc/meminfo | grep MemAvailable

# 检查进程资源限制
cat /proc/{pid}/limits | grep "Max open files"

# 检查 IPC 服务状态
ps -A | grep ipc
```

**解决**:
1. 释放系统内存
2. 重启应用/服务
3. 检查底层 IPC 服务状态

---

### 问题 5: 数据写入失败（1900009）

**现象**:
```
BusinessException: code=1900009, message="Write data to message sequence failed."
```

**代码位置**: `ohos/rpc/message_sequence.cj:66-189`

**常见原因**:

1. **容量不足**:
```cangjie
// 默认容量 200KB
ms.setCapacity(1024)  // 设置容量为 1KB
ms.writeString(largeString)  // 写入 2KB 数据 -> 失败
```

2. **位置错误**:
```cangjie
ms.rewindWrite(999999)  // 无效位置 -> 失败
```

3. **空数组**:
```cangjie
ms.writeUInt8Array([])  // 空数组 -> 失败
```

**诊断**:
```cangjie
// 检查容量和大小
let capacity = ms.getCapacity()
let size = ms.getSize()
let writable = ms.getWritableBytes()
println("Capacity: ${capacity}, Size: ${size}, Writable: ${writable}")
```

**解决**:
1. 使用 `setCapacity()` 增加容量
2. 检查写入位置
3. 验证数组非空

---

### 问题 6: Ashmem 映射失败（1900001）

**现象**:
```
BusinessException: code=1900001, message="Call mmap function failed."
```

**代码位置**: `ohos/rpc/ashmem.cj:168-186`

```cangjie
public func mapTypedAshmem(mapType: UInt32): Unit {
    var errCode = 0i32
    unsafe { FfiRpcAshmemImplMapTypedAshmem(getID(), mapType, inout errCode) }
    checkAndThrow(errCode)  // 1900001
}
```

**原因**:
1. 虚拟内存不足
2. 权限不足（需要 CAP_IPC_LOCK）
3. 无效的 Ashmem 对象

**诊断**:
```bash
# 检查虚拟内存
vmstat -s | grep "free memory"

# 检查进程权限
cat /proc/{pid}/status | grep Cap

# 检查 Ashmem 对象有效性
ls -la /dev/ashmem/
```

**解决**:
1. 减小 Ashmem 大小
2. 确保应用有适当权限
3. 重新创建 Ashmem 对象

---

### 问题 7: IPC 通信失败（1900007）

**现象**:
```
BusinessException: code=1900007, message="Communication failed."
```

**可能原因**:
1. 目标服务未运行
2. 权限不足
3. 网络问题（RPC 跨设备）

**诊断**:
```bash
# 检查目标服务
ps -A | grep {service_name}

# 检查日志
hilog | grep -i "ipc\|rpc\|communication"

# 检查网络（跨设备 RPC）
ping {remote_device_ip}
```

**解决**:
1. 启动目标服务
2. 检查应用权限配置
3. 确保设备网络连通

---

### 问题 8: 文件描述符传递失败

**现象**:
```
BusinessException: code=1900013, message="Call os dup function failed."
```

**代码位置**: `ohos/rpc/message_sequence.cj:1028-1033`

```cangjie
public static func dupFileDescriptor(fd: Int32): Int32 {
    let dupFd = unsafe { FfiRpcMessageSequenceImplDupFileDescriptor(fd) }
    if (dupFd < 0) {
        throw BusinessException(ErrorCode.OsDupError.value, ...)
    }
    return dupFd
}
```

**原因**:
1. FD 无效（负数或未打开）
2. 达到进程 FD 限制

**诊断**:
```bash
# 检查进程 FD 使用情况
ls -la /proc/{pid}/fd/ | wc -l
cat /proc/{pid}/limits | grep "Max open files"

# 检查 FD 有效性
ls -la /proc/{pid}/fd/{fd_number}
```

**解决**:
1. 使用有效的 FD
2. 关闭不必要的 FD
3. 增加 FD 限制

---

### 问题 9: Parcelable 序列化失败

**现象**: `writeParcelable()` 抛出异常或返回失败

**代码位置**: `ohos/rpc/message_sequence.cj:1319-1328`

```cangjie
public func writeParcelable<T>(val: T): Unit where T <: Parcelable {
    let pos = getWritePosition()
    writeInt(1)
    try {
        val.marshalling(this)
    } catch (e: BusinessException) {
        rewindWrite(pos)
        checkAndThrow(ErrorCode.WriteDataToMessageSequenceError.value)
    }
}
```

**原因**:
1. `marshalling()` 实现抛出异常
2. 写入数据超出容量
3. 数据格式不匹配

**诊断**:
```cangjie
// 添加调试日志
class MyData <: Parcelable {
    public func marshalling(dataOut: MessageSequence): Bool {
        try {
            dataOut.writeInt(this.id)
            dataOut.writeString(this.name)
            return true
        } catch (e: BusinessException) {
            // 打印详细错误
            println("Marshalling failed: ${e.code}, ${e.message}")
            return false
        }
    }
}
```

**解决**:
1. 确保 `marshalling()` 正确处理异常
2. 检查 MessageSequence 容量
3. 验证数据类型匹配

---

## 调试技巧

### 启用详细日志

```cangjie
import ohos.hilog.*

// 使用 HiLog 打印调试信息
HiLog.info(label: "IPC_TEST", "MessageSequence created, id: ${ms.getID()}")
```

### 检查 FFI 调用结果

```cangjie
// 包装 FFI 调用以添加调试
func debugFFICall<T>(name: String, operation: ()->T): T {
    println("[FFI] Calling: ${name}")
    let result = operation()
    println("[FFI] ${name} completed")
    return result
}
```

### 验证 MessageSequence 状态

```cangjie
func verifyMessageSequence(ms: MessageSequence): Bool {
    try {
        let capacity = ms.getCapacity()
        let size = ms.getSize()
        let writable = ms.getWritableBytes()
        let readable = ms.getReadableBytes()
        
        println("MS State: capacity=${capacity}, size=${size}")
        println("MS State: writable=${writable}, readable=${readable}")
        
        // 验证一致性
        return (size <= capacity) && (writable == capacity - size)
    } catch (e: BusinessException) {
        println("MS Verification failed: ${e.message}")
        return false
    }
}
```

### Ashmem 调试

```cangjie
func debugAshmem(ashmem: Ashmem): Unit {
    try {
        let size = ashmem.getAshmemSize()
        println("Ashmem size: ${size}")
        
        // 测试读写
        let testData = Array<Byte>(1024, repeat: 0xAB)
        ashmem.mapReadWriteAshmem()
        ashmem.writeDataToAshmem(testData, 1024, 0)
        let readData = ashmem.readDataFromAshmem(1024, 0)
        println("Ashmem read/write test: ${readData[0] == 0xAB}")
        ashmem.unmapAshmem()
    } catch (e: BusinessException) {
        println("Ashmem debug failed: ${e.code}, ${e.message}")
    }
}
```

---

## 性能优化建议

### 1. 避免频繁创建 MessageSequence

```cangjie
// 不推荐：频繁创建
for (i in 0..1000) {
    let ms = MessageSequence.create()
    // ...
    ms.reclaim()
}

// 推荐：复用
let ms = MessageSequence.create()
for (i in 0..1000) {
    ms.setSize(0)  // 重置
    // ...
}
ms.reclaim()
```

### 2. 大数据使用 Ashmem

```cangjie
// 超过 200KB 使用 Ashmem
if (data.size > 200 * 1024) {
    let ashmem = Ashmem.create("data", data.size)
    ashmem.mapReadWriteAshmem()
    ashmem.writeDataToAshmem(data, data.size, 0)
    ms.writeAshmem(ashmem)
} else {
    ms.writeByteArray(data)
}
```

### 3. 预分配容量

```cangjie
// 预先计算所需容量
let estimatedSize = calculateSize(data)
let ms = MessageSequence.create()
ms.setCapacity(estimatedSize)
// 批量写入
```

---

## 参考文档

- [错误码附录](appendix/Error_Codes.md) - 完整错误码列表
- [N-API 参考](03_NAPI_Reference.md) - API 详细说明
- [安全风险评审](06_Security_Analysis.md) - 安全相关注意事项
