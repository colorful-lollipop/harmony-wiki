# 内部实现细节

**本文档深入讲解核心实现机制，适合需要理解内部工作原理的开发者。**

---

## 1. FFI 绑定机制详解

### 1.1 Foreign 函数声明

**文件**: `ohos/common_event_manager/common_event_manager_ffi.cj`

Cangjie 使用 `foreign` 块声明外部 C/C++ 函数：

```cangjie
foreign {
    // 发布事件（基础版）
    func CJ_PublishEvent(event: CString, userId: Int32): Int32
    
    // 发布事件（带完整数据）
    func CJ_PublishEventWithData(
        event: CString, 
        userId: Int32, 
        options: CCommonEventPublishData
    ): Int32
    
    // 订阅者管理
    func FfiCommonEventManagerCreateSubscriber(
        info: CPointer<CSubscribeInfo>, 
        errorCode: CPointer<Int32>
    ): Int64
    
    func CJ_Subscribe(id: Int64, funcId: Int64): Int32
    func CJ_Unsubscribe(id: Int64): Int32
}
```

**关键机制**:
- `CString`: C 空终止字符串，需手动管理内存
- `CPointer<T>`: C 指针类型，用于传递结构体地址
- `Int64` 句柄: FFI 层返回的资源标识符
- `Int32` 返回值: 错误码，0 表示成功

### 1.2 C 兼容结构体

使用 `@C` 注解定义 C 内存布局兼容的结构体：

```cangjie
@C
protected struct CCommonEventPublishData {
    var bundleName: CString = CString(CPointer())
    var data: CString = CString(CPointer())
    let code: Int32
    var permissions: CArrString = CArrString(CPointer<CString>(), 0)
    let isOrdered: Bool
    let isSticky: Bool
    var parameters: CArrParameters = CArrParameters(CPointer<CParameters>(), 0)
}
```

**内存布局要求**:
1. 字段顺序必须与 C 结构体一致
2. 使用 C 兼容类型 (`CString`, `CPointer`, `Int32`, `Bool`)
3. 变长数据使用指针+长度形式 (`CArrString`, `CArrParameters`)

### 1.3 调用约定

**同步调用**:
```cangjie
unsafe {
    let retCode = CJ_PublishEventWithData(cEvent.value, UNDEFINED_USER, cOptions.value)
    throwIfNotSuccess(retCode, "publish")
}
```

**异步回调**:
```cangjie
// 1. 定义回调包装器
let wrapper = { value: CCommonEventData =>
    let commonData = CommonEventData(value)
    callback(None, commonData)
}

// 2. 创建 FFI 回调对象
let lambdaData = Callback1Param<CCommonEventData, Unit>(wrapper)

// 3. 传递回调 ID
let retCode = unsafe { CJ_Subscribe(subscriber.getID(), lambdaData.getID()) }
```

---

## 2. 内存管理机制

### 2.1 字符串内存生命周期

**分配**:
```cangjie
// LibC.mallocCString 分配 C 字符串内存
let cString = LibC.mallocCString("event_name")
```

**释放策略**:

| 策略 | 使用场景 | 示例 |
|------|----------|------|
| RAII (推荐) | 临时字符串 | `try (cEvent = LibC.mallocCString(event).asResource())` |
| 手动 free | 结构体内字段 | `CCommonEventPublishData.free()` |
| 析构自动释放 | FFI 句柄 | `~init() { releaseFFIData(id) }` |

**RAII 模式详解**:
```cangjie
unsafe {
    try (
        cEvent = LibC.mallocCString(event).asResource(),
        cOptions = CCommonEventPublishData(options).asResource()
    ) {
        // 使用资源
        CJ_PublishEventWithData(cEvent.value, UNDEFINED_USER, cOptions.value)
    }
    // 自动调用资源的 free 方法
}
```

### 2.2 结构体内存管理

**CCommonEventPublishData 生命周期**:
```cangjie
protected init(c: CommonEventPublishData) {
    unsafe {
        try {
            // 分配所有字符串字段
            this.bundleName = LibC.mallocCString(c.bundleName)
            this.data = LibC.mallocCString(c.data)
            // ...
        } catch (e: Exception) {
            // 异常时清理已分配资源
            free()
            throw BusinessException(1500009, "Error obtaining system parameters.")
        }
    }
}

func free(): Unit {
    unsafe {
        LibC.free(bundleName)
        LibC.free(data)
        // 释放权限数组
        for (i in 0..permissions.size) {
            LibC.free(permissions.head.read(i))
        }
        LibC.free(permissions.head)
        parameters.free()
    }
}
```

**异常安全原则**:
1. 构造函数中异常必须释放已分配资源
2. 批量分配时，失败时清理部分完成的资源
3. 使用 `try-catch` 包裹所有可能失败的分配操作

### 2.3 FFI 句柄管理

**RemoteDataLite 模式**:
```cangjie
public class CommonEventSubscriber <: RemoteDataLite {
    protected init(id: Int64) {
        super(id)  // 保存 FFI 句柄
    }
    
    ~init() {
        // 析构时通知 FFI 层释放资源
        releaseFFIData(myDataId)
    }
}
```

**生命周期保证**:
- Cangjie 对象存活期间，FFI 句柄有效
- GC 回收时自动调用析构函数
- 即使 callback 异常，句柄最终会被释放

---

## 3. 数据序列化机制

### 3.1 参数类型系统

**CommonEventValueType 枚举**:
```cangjie
public enum CommonEventValueType {
    | Int32Value(Int32)
    | Float64Value(Float64)
    | StringValue(String)
    | BoolValue(Bool)
    | FD(Int32)
    | ArrayString(Array<String>)
    | ArrayInt32(Array<Int32>)
    // ... 更多数组类型
}
```

**类型标记常量**:
```cangjie
const INT_TYPE: Int8 = 0
const F64_TYPE: Int8 = 1
const STRING_TYPE: Int8 = 2
const BOOL_TYPE: Int8 = 3
const FD_TYPE: Int8 = 4
// ... 共 11 种类型
```

### 3.2 CParameters 结构体

**C 侧表示**:
```cangjie
@C
protected struct CParameters {
    let valueType: Int8      // 类型标记
    let key: CString         // 参数名
    let value: CPointer<Unit> // 类型擦除的值指针
    let size: Int64          // 数组长度（标量为 1）
}
```

### 3.3 序列化流程

**Cangjie → C 转换** (`getValue` 函数):
```cangjie
func getValue(value: CommonEventValueType): (Int8, CPointer<Unit>, Int64) {
    unsafe {
        match (value) {
            case Int32Value(v) => 
                return (INT_TYPE, createPtr<Int32>(v), 1)
            case StringValue(v) =>
                let ptr = LibC.mallocCString(v).getChars()
                return (STRING_TYPE, CPointer<Unit>(ptr), 1)
            case ArrayString(v) =>
                throwIfEmpty(v)
                let ptr = createCpCString(v)
                return (ARRSTRING_TYPE, CPointer<Unit>(ptr), v.size)
            // ... 其他类型
        }
    }
}
```

**数组处理**:
```cangjie
unsafe func createArrPtr<T>(value: Array<T>): CPointer<Unit> where T <: CType {
    let ptr = safeMalloc<T>(count: value.size)
    for (i in 0..value.size) {
        ptr.write(i, value[i])
    }
    return CPointer<Unit>(ptr)
}
```

### 3.4 反序列化流程

**C → Cangjie 转换** (`Parameters.init`):
```cangjie
protected init(c: CParameters) {
    this.key = c.key.toString()
    this.value = unsafe {
        match {
            case c.valueType == INT_TYPE => 
                Int32Value(CPointer<Int32>(c.value).read())
            case c.valueType == STRING_TYPE => 
                StringValue(CString(CPointer<UInt8>(c.value)).toString())
            case c.valueType == ARRSTRING_TYPE => 
                ArrayString(c.toArrString())
            // ... 其他类型
            case _ => ArrayFD(c.toArr<Int32>())  // 默认回退
        }
    }
}
```

**数组还原**:
```cangjie
func toArrString(): Array<String> {
    unsafe {
        Array<String>(
            size,
            { i =>
                let ptr = CPointer<CString>(value).read(i)
                return ptr.toString()
            }
        )
    }
}
```

---

## 4. 错误处理机制

### 4.1 错误码定义

**文件**: `common_event_manager_errors.cj`

```cangjie
const ERROR_NULL_ACTION: Int32 = 1500001
const ERROR_SANDBOX_APPLICATION: Int32 = 1500002
const ERROR_FREQUENCY_TOO_HIGH: Int32 = 1500003
// ... 共 10 个错误码

let ERROR_CODE_MAP = HashMap<Int32, String>(
    (ERROR_FREQUENCY_TOO_HIGH, "The common event sending frequency too high."),
    (ERROR_SENDING_MSG_TO_CES, "Error sending message to Common Event Service."),
    // ...
)
```

### 4.2 错误码转换

**映射逻辑**:
```cangjie
func getErrorCode(code: Int32): Int32 {
    const MEMORY_ERROR: Int32 = -2
    const ERROR_CES_FAILED: Int32 = 1
    if (code == MEMORY_ERROR || code == INVALID_CODE || code == ERROR_CES_FAILED) {
        ERROR_CES_UNINITIALIZED  // 统一映射为初始化错误
    } else {
        code
    }
}
```

### 4.3 异常抛出

**统一错误处理函数**:
```cangjie
func throwIfNotSuccess(code: Int32, funcName: String): Unit {
    if (code != SUCCESS_CODE) {
        let errCode = getErrorCode(code)
        throw BusinessException(errCode, "${funcName} failed: ${getErrorMsg(errCode)}")
    }
}
```

**使用示例**:
```cangjie
let retCode = unsafe { CJ_PublishEventWithData(...) }
throwIfNotSuccess(retCode, "publish")  // 失败时抛出 BusinessException
```

---

## 5. 线程模型

### 5.1 API 线程要求

| API | 执行线程 | 声明位置 |
|-----|---------|----------|
| `publish` | worker thread | `@!APILevel[workerthread: true]` |
| `createSubscriber` | worker thread | `@!APILevel[workerthread: true]` |
| `subscribe` | main thread | 无 workerthread 标记 |
| `unsubscribe` | worker thread | `@!APILevel[workerthread: true]` |

### 5.2 回调执行机制

**subscribe 回调在主线程执行**:
```cangjie
public static func subscribe(
    subscriber: CommonEventSubscriber,
    callback: AsyncCallback<CommonEventData>
): Unit {
    // 包装回调，转换数据类型
    let wrapper = { value: CCommonEventData =>
        let commonData = CommonEventData(value)
        callback(None, commonData)  // 在主线程调用
    }
    
    let lambdaData = Callback1Param<CCommonEventData, Unit>(wrapper)
    let retCode = unsafe { CJ_Subscribe(subscriber.getID(), lambdaData.getID()) }
    throwIfNotSuccess(retCode, "subscribe")
}
```

**为什么 subscribe 必须在主线程？**
- 回调需要更新 UI
- 保证事件处理顺序
- 避免多线程竞争

---

## 6. 系统事件常量机制

### 6.1 Support 类结构

**文件**: `support.cj` (约 51KB，150+ 常量)

```cangjie
public class Support {
    // 系统启动事件
    public static const COMMON_EVENT_BOOT_COMPLETED: String = "usual.event.BOOT_COMPLETED"
    public static const COMMON_EVENT_LOCKED_BOOT_COMPLETED: String = "usual.event.LOCKED_BOOT_COMPLETED"
    
    // 电源事件
    public static const COMMON_EVENT_BATTERY_CHANGED: String = "usual.event.BATTERY_CHANGED"
    public static const COMMON_EVENT_BATTERY_LOW: String = "usual.event.BATTERY_LOW"
    
    // ... 150+ 个常量
}
```

### 6.2 事件分类

| 分类 | 前缀 | 数量 | 示例 |
|------|------|------|------|
| 系统生命周期 | `usual.event.BOOT_*` | ~5 | BOOT_COMPLETED |
| 电源管理 | `usual.event.BATTERY_*` | ~10 | BATTERY_CHANGED |
| 包管理 | `usual.event.PACKAGE_*` | ~8 | PACKAGE_ADDED |
| WiFi | `usual.event.wifi.*` | ~12 | WIFI_POWER_STATE |
| 蓝牙 | `usual.event.bluetooth.*` | ~15 | BLUETOOTH_HOST_STATE_UPDATE |
| NFC | `usual.event.nfc.*` | ~6 | NFC_ACTION_ADAPTER_STATE_CHANGED |
| 用户管理 | `usual.event.USER_*` | ~8 | USER_STARTED |
| 显示 | `usual.event.SCREEN_*` | ~4 | SCREEN_ON |
| 存储 | `usual.event.DISK_*` | ~6 | DISK_REMOVED |

---

## 7. 边界检查与安全机制

### 7.1 已实现的检查

**数组非空检查**:
```cangjie
func throwIfEmpty<T>(arr: Array<T>): Unit {
    if (arr.size == 0) {
        throw BusinessException(ERR_PARAMETER_ERROR, "Parameter error.")
    }
}
```

**优先级范围限制**:
```cangjie
func checkPriority(priority: Int32): Int32 {
    let maxValue = 1000i32
    let minValue = -100i32
    if (priority > maxValue) { maxValue }
    else if (priority < minValue) { minValue }
    else { priority }
}
```

### 7.2 缺失的检查（安全风险）

| 检查项 | 状态 | 风险 |
|--------|------|------|
| 字符串长度限制 | ❌ 缺失 | 内存溢出 |
| HashMap 大小限制 | ❌ 缺失 | DoS |
| userId 有效性 | ❌ 缺失 | 无效操作 |
| FD 类型权限 | ❌ 缺失 | 权限提升 |
| 粘性事件权限 | ⚠️ 仅注解 | 权限绕过 |

详见 [05_Security.md](05_Security.md)。

---

## 8. 调试与日志

### 8.1 日志输出

**依赖**: `hiviewdfx_cangjie_wrapper:ohos.hilog`

```cangjie
import ohos.hilog.HiLog

// 日志输出示例
HiLog.info(LogTag.COMMON_EVENT, "Creating subscriber with %{public}s", events)
```

### 8.2 调试要点

1. **FFI 调用失败**: 检查错误码映射
2. **内存泄漏**: 验证 `free()` 是否被调用
3. **类型转换错误**: 检查 `valueType` 标记
4. **订阅无响应**: 确认线程模型和回调注册

---

## 9. 性能考虑

### 9.1 内存分配优化

| 场景 | 优化策略 |
|------|----------|
| 频繁发布事件 | 复用 CommonEventPublishData 对象 |
| 大参数列表 | 限制 parameters HashMap 大小 |
| 字符串操作 | 避免重复 malloc/free |

### 9.2 序列化开销

- 简单类型（Int32, Bool）: O(1)
- 字符串: O(n)，n 为字符串长度
- 数组: O(n*m)，n 为数组长度，m 为元素大小
- HashMap: O(k*n)，k 为键值对数量

---

**相关文档**:
- [架构设计 →](02_Architecture.md)
- [代码地图 →](03_CodeMap.md)
- [API 参考 →](03_API_Reference.md)
- [安全评审 →](05_Security.md)
