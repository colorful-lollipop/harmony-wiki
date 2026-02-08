# 安全风险评审

## 目的

本文档基于代码证据对 `communication_cangjie_wrapper` 进行安全风险分析，识别攻击面、信任边界和可被利用点。

## 适用范围

- 安全审计人员
- 安全架构师
- 代码审查人员

## 评估方法

- **检查范围**: `ohos/rpc/*.cj`（不含测试）
- **评估维度**: 输入验证、内存安全、权限控制、资源管理、序列化安全
- **证据标准**: 代码路径 + 符号名 + 行号

## 攻击面清单

### 1. 外部输入接口

| 接口类型 | 方法数量 | 风险等级 | 证据 |
|----------|----------|----------|------|
| MessageSequence 写入 | 40+ | 中 | message_sequence.cj:95-1437 |
| Ashmem 创建/写入 | 5 | 高 | ashmem.cj:89-271 |
| Parcelable 序列化 | 2 | 中 | parcelable.cj:42-54 |
| 文件描述符传递 | 4 | 高 | message_sequence.cj:1013-1083 |

### 2. 攻击面分类

```
┌─────────────────────────────────────────────────────────────────┐
│                        攻击面分析                                │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  外部输入 ───┬──> MessageSequence.write*() ───> FFI ───> 底层 │
│             │        - 基本类型                                  │
│             │        - 数组                                      │
│             │        - 字符串                                    │
│             │        - Parcelable (用户自定义)                   │
│             │                                                    │
│             ├──> Ashmem.create()/write*() ───> 共享内存        │
│             │        - 大小参数                                  │
│             │        - 偏移参数                                  │
│             │        - 原始数据                                  │
│             │                                                    │
│             ├──> 文件描述符传递 ───> FD 传输                    │
│             │        - writeFileDescriptor()                     │
│             │        - dupFileDescriptor()                       │
│             │                                                    │
│             └──> 原始数据缓冲区 ───> 大块内存操作                │
                      - writeRawDataBuffer()                       │
                      - readRawDataBuffer()                        │
└─────────────────────────────────────────────────────────────────┘
```

## 信任边界

### 边界 1: Cangjie ↔ FFI 边界

```
┌─────────────────┐     FFI      ┌─────────────────┐
│   Cangjie 层     │  ──────────>  │   C/C++ 层      │
│                 │   类型转换    │                 │
│  - 参数校验      │              │  - 系统调用      │
│  - 异常处理      │              │  - 驱动交互      │
└─────────────────┘              └─────────────────┘
      ↑ 本组件责任边界                ↑ 外部组件责任
```

### 边界 2: 应用 ↔ 系统服务边界

```
┌─────────────────┐                ┌─────────────────┐
│   应用进程       │  IPC/RPC       │   系统服务       │
│                 │  ────────────> │                 │
│  - 用户代码      │                │  - 特权操作      │
│  - 不可信输入    │                │  - 受信处理      │
└─────────────────┘                └─────────────────┘
      ↑ 不可信                           ↑ 可信
```

## 可被利用点分析

### 可被利用点 1: Ashmem 大小参数未充分校验

**证据**: `ohos/rpc/ashmem.cj:89-100`

```cangjie
public static func create(name: String, size: Int32): Ashmem {
    if (name.isEmpty()) {
        throw BusinessException(ErrorCode.CheckParamError.value, "Parameter error.")
    }
    // size 参数直接传递到 FFI，无范围校验
    let id = unsafe { FfiRpcAshmemImplCreate(cname.value, size) }
    ...
}
```

**问题**:
- `size` 参数仅经过 FFI 传递，无显式范围校验
- 负数 `size` 可能导致未定义行为
- 极大 `size` 可能导致内存耗尽

**触发路径**:
```
攻击者调用 Ashmem.create("x", -1) 或极大值
    -> FFI -> 底层 mmap
        -> 内核错误/内存耗尽
```

**影响**: 拒绝服务 (DoS)、潜在的内存破坏

**修复建议**:
```cangjie
public static func create(name: String, size: Int32): Ashmem {
    if (name.isEmpty()) {
        throw BusinessException(ErrorCode.CheckParamError.value, "Parameter error.")
    }
    // 添加 size 校验
    if (size <= 0 || size > MAX_ASHMEM_SIZE) {
        throw BusinessException(ErrorCode.CheckParamError.value, 
            "Invalid ashmem size: must be 1..MAX_ASHMEM_SIZE")
    }
    ...
}
```

**当前状态**: ⚠️ 存在风险

---

### 可被利用点 2: Ashmem 偏移参数未校验

**证据**: `ohos/rpc/ashmem.cj:233-244`

```cangjie
public func writeDataToAshmem(buf: Array<Byte>, size: Int64, offset: Int64): Unit {
    if (buf.size == 0) {
        throw BusinessException(ErrorCode.WriteToAshmemError.value, "Write to ashmem failed.")
    }
    // size 和 offset 直接传递到 FFI，无校验
    FfiRpcAshmemImplWriteDataToAshmem(getID(), cp.pointer, size, offset, inout errCode)
    ...
}
```

**问题**:
- `size` 和 `offset` 无范围校验
- 负值、越界值可能导致内存破坏
- `size` 与 `buf.size` 的关系未强制校验

**触发路径**:
```
攻击者调用 writeDataToAshmem(buf, largeSize, negativeOffset)
    -> FFI -> 底层 memcpy
        -> 内存越界写入
```

**影响**: 内存破坏、信息泄露、潜在的代码执行

**修复建议**:
```cangjie
public func writeDataToAshmem(buf: Array<Byte>, size: Int64, offset: Int64): Unit {
    if (buf.size == 0) {
        throw BusinessException(ErrorCode.WriteToAshmemError.value, "Write to ashmem failed.")
    }
    // 添加参数校验
    if (size <= 0 || size > buf.size) {
        throw BusinessException(ErrorCode.CheckParamError.value, "Invalid size")
    }
    if (offset < 0) {
        throw BusinessException(ErrorCode.CheckParamError.value, "Invalid offset")
    }
    // 校验 offset + size <= ashmemSize
    let ashmemSize = getAshmemSize()
    if (offset + size > ashmemSize) {
        throw BusinessException(ErrorCode.CheckParamError.value, "Write exceeds ashmem size")
    }
    ...
}
```

**当前状态**: ⚠️ 存在风险

---

### 可被利用点 3: 原始数据缓冲区大小未充分校验

**证据**: `ohos/rpc/message_sequence.cj:1413-1424`

```cangjie
public func writeRawDataBuffer(rawData: Array<Byte>, size: Int64): Unit {
    if (size <= 0 || size > rawData.size) {
        throw BusinessException(ErrorCode.CheckParamError.value, "Parameter error.")
    }
    // 校验通过，但 size 可能仍然极大
    FfiRpcMessageSequenceImplWriteRawDataBuffer(getID(), cp.pointer, size, inout errCode)
    ...
}
```

**问题**:
- 虽然检查了 `size > rawData.size`，但未限制最大 size
- 极大 `size` 可能导致内存分配失败或性能问题

**触发路径**:
```
攻击者构造极大的 rawData 数组和 size
    -> 触发大内存分配
        -> DoS
```

**影响**: 拒绝服务 (DoS)

**修复建议**:
```cangjie
const MAX_RAW_DATA_SIZE: Int64 = 128 * 1024 * 1024  // 128MB

public func writeRawDataBuffer(rawData: Array<Byte>, size: Int64): Unit {
    if (size <= 0 || size > rawData.size) {
        throw BusinessException(ErrorCode.CheckParamError.value, "Parameter error.")
    }
    if (size > MAX_RAW_DATA_SIZE) {
        throw BusinessException(ErrorCode.CheckParamError.value, 
            "Raw data size exceeds maximum (128MB)")
    }
    ...
}
```

**当前状态**: ⚠️ 存在风险

---

### 可被利用点 4: 文件描述符传递无有效性检查

**证据**: `ohos/rpc/message_sequence.cj:1062-1083`

```cangjie
public func writeFileDescriptor(fd: Int32): Unit {
    var errCode: Int32 = 0
    unsafe { FfiRpcMessageSequenceImplWriteFileDescriptor(getID(), fd, inout errCode) }
    checkAndThrow(errCode)
}

public static func dupFileDescriptor(fd: Int32): Int32 {
    let dupFd = unsafe { FfiRpcMessageSequenceImplDupFileDescriptor(fd) }
    if (dupFd < 0) {
        throw BusinessException(ErrorCode.OsDupError.value, ERR_CODE_MAP[ErrorCode.OsDupError.value])
    }
    return dupFd
}
```

**问题**:
- `fd` 参数无有效性检查（负数、未打开的 FD）
- 依赖于底层返回错误，但错误处理可能不充分
- 恶意 FD 可能导致安全问题

**触发路径**:
```
攻击者传递恶意构造的 FD
    -> FFI -> 系统调用
        -> 未定义行为或安全漏洞
```

**影响**: 信息泄露、权限提升（取决于 FD 类型）

**修复建议**:
```cangjie
public func writeFileDescriptor(fd: Int32): Unit {
    if (fd < 0) {
        throw BusinessException(ErrorCode.CheckParamError.value, "Invalid file descriptor")
    }
    // 可选：检查 FD 是否实际打开
    ...
}
```

**当前状态**: ⚠️ 存在风险

---

### 可被利用点 5: Parcelable 自定义序列化安全风险

**证据**: `ohos/rpc/message_sequence.cj:1319-1399`

```cangjie
public func writeParcelable<T>(val: T): Unit where T <: Parcelable {
    let pos = getWritePosition()
    writeInt(1)
    try {
        val.marshalling(this)  // 调用用户提供的代码
    } catch (e: BusinessException) {
        rewindWrite(pos)
        checkAndThrow(ErrorCode.WriteDataToMessageSequenceError.value)
    }
}

public func readParcelable<T>(dataIn: T): Unit where T <: Parcelable {
    let len = readInt()
    if (len > 0) {
        dataIn.unmarshalling(this)  // 调用用户提供的代码
        return
    }
    checkAndThrow(ErrorCode.ReadDataFromMessageSequenceError.value)
}
```

**问题**:
- `marshalling`/`unmarshalling` 由用户实现
- 用户代码可能存在安全问题
- 反序列化时无类型验证

**触发路径**:
```
攻击者提供恶意 Parcelable 实现
    -> unmarshalling() 执行攻击者代码
        -> 安全漏洞
```

**影响**: 代码执行、信息泄露、权限提升

**缓解措施**:
1. 文档明确警告开发者安全实现要求
2. 沙箱限制 Parcelable 实现权限
3. 代码签名验证

**当前状态**: ⚠️ 设计层面风险，需开发者注意

---

### 可被利用点 6: CStringArray 构造时内存泄漏

**证据**: `ohos/rpc/request_result.cj:340-383`

```cangjie
init(arr: Array<String>) {
    if (arr.size == 0) {
        this.data = CPointer<CString>()
        this.len = 0
    } else {
        this.data = safeMalloc<CString>(count: arr.size)
        for (i in 0..arr.size) {
            try {
                unsafe { data.write(i, LibC.mallocCString(arr[i])) }
            } catch (e: Exception) {
                for (j in 0..i) {
                    unsafe { LibC.free(data.read(j)) }
                }
                unsafe { LibC.free(data) }
                throw e
            }
        }
        this.len = UInt32(arr.size)
    }
}
```

**问题**:
- 异常处理正确，但依赖 `Exception` 捕获
- `LibC.mallocCString` 可能因超长字符串失败
- 无单个字符串长度限制

**触发路径**:
```
攻击者提供包含超长字符串的数组
    -> 内存分配失败
        -> 服务中断
```

**影响**: 拒绝服务 (DoS)

**修复建议**:
```cangjie
const MAX_STRING_LENGTH: UIntNative = 1024 * 1024  // 1MB

init(arr: Array<String>) {
    for (s in arr) {
        if (s.size > MAX_STRING_LENGTH) {
            throw BusinessException(ErrorCode.CheckParamError.value, "String too long")
        }
    }
    ...
}
```

**当前状态**: ✅ 异常处理正确，但建议添加长度限制

---

### 可被利用点 7: FFI 边界类型混淆风险

**证据**: `ohos/rpc/cj_rpc_ffi.cj`

FFI 函数使用 C 指针类型：
```cangjie
foreign func FfiRpcMessageSequenceImplWriteArrayBuffer(
    id: Int64, 
    typeCode: Int32, 
    value: CPointer<Unit>,  // 泛型指针
    byteLength: UIntNative, 
    errCode: CPointer<Int32>
): Unit
```

**问题**:
- `CPointer<Unit>` 是泛型指针，类型安全依赖调用方
- 错误的 `typeCode` 与数据类型匹配可能导致类型混淆

**触发路径**:
```
内部实现错误传递 typeCode
    -> 类型混淆
        -> 内存解释错误
```

**影响**: 内存破坏、信息泄露

**缓解措施**:
- 代码审查确保 typeCode 与数据类型匹配
- 运行时类型检查（如可能）

**当前状态**: ✅ 当前实现正确，需维护时注意

---

## 安全建议汇总

### 高优先级修复

1. **Ashmem 参数校验** (可被利用点 1, 2)
   - 添加 `size` 和 `offset` 范围校验
   - 防止负值和越界访问

2. **原始数据大小限制** (可被利用点 3)
   - 限制单次传输最大大小
   - 与 `getRawDataCapacity()` 一致

### 中优先级修复

3. **文件描述符校验** (可被利用点 4)
   - 添加 FD 有效性检查
   - 考虑 FD 类型验证

4. **字符串长度限制** (可被利用点 6)
   - 限制单个字符串最大长度
   - 限制字符串数组总大小

### 设计层面注意

5. **Parcelable 安全** (可被利用点 5)
   - 文档明确安全要求
   - 考虑运行时沙箱

6. **FFI 类型安全** (可被利用点 7)
   - 代码审查流程
   - 自动化类型检查

## 局限性说明

### 本评审未覆盖的内容

1. **底层 C/C++ 实现**: `ipc:cj_ipc_ffi` 的内部实现未审查
2. **Binder/SoftBus 驱动**: 内核层安全由 OS 保障
3. **权限系统**: 依赖 OpenHarmony 的 Access Token 机制
4. **测试代码**: 按约束要求未审查

### 需要进一步确认的内容

1. TODO: `ipc:cj_ipc_ffi` 的输入校验策略
2. TODO: OpenHarmony IPC 权限模型的具体实现
3. TODO: Binder 驱动的安全边界

## 参考文档

- [N-API 参考](03_NAPI_Reference.md) - API 详细参数
- [内部 API](04_Internal_API.md) - 内部接口实现
- [目录结构](02_Directory_Structure.md) - 代码文件位置
