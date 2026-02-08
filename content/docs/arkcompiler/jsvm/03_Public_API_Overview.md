# 对外 API 总览

> JSVM-API 高层视图与分类

## 目的与适用范围

### 目的
本文档提供 JSVM 对外 API 的高层视图，帮助开发者快速了解 API 面貌和分类。

### 适用范围
- 需要了解 JSVM-API 整体结构的开发者
- 需要选择合适 API 的开发者
- 需要进行 API 审查的开发者

---

## API 分类

JSVM-API 按功能分为以下几大类：

### 1. VM 生命周期管理（VM Lifecycle）

**功能**: 管理 VM、Environment、Scope 的创建与销毁。

**API 数量**: ~15 个

**主要 API**:
- `OH_JSVM_Init()` - 初始化 JSVM 全局状态
- `OH_JSVM_CreateVM()` - 创建 VM 实例
- `OH_JSVM_DestroyVM()` - 销毁 VM 实例
- `OH_JSVM_CreateEnv()` - 创建 Environment
- `OH_JSVM_DestroyEnv()` - 销毁 Environment
- `OH_JSVM_CreateEnvFromSnapshot()` - 从快照创建 Environment
- `OH_JSVM_OpenVMScope()` / `OH_JSVM_CloseVMScope()` - VM Scope 管理
- `OH_JSVM_OpenEnvScope()` / `OH_JSVM_CloseEnvScope()` - Env Scope 管理
- `OH_JSVM_OpenHandleScope()` / `OH_JSVM_CloseHandleScope()` - Handle Scope 管理

**证据位置**: `interface/kits/jsvm.h:323-476`

### 2. 代码编译与执行（Code Compilation & Execution）

**功能**: 编译 JavaScript 代码、执行脚本、生成代码缓存。

**API 数量**: ~10 个

**主要 API**:
- `OH_JSVM_CompileScript()` - 编译脚本
- `OH_JSVM_CompileScriptWithOrigin()` - 带源信息编译脚本
- `OH_JSVM_CompileScriptWithOptions()` - 带选项编译脚本
- `OH_JSVM_RunScript()` - 执行脚本
- `OH_JSVM_CreateCodeCache()` - 创建代码缓存
- `OH_JSVM_CreateSnapshot()` - 创建快照
- `OH_JSVM_JsonParse()` / `OH_JSVM_JsonStringify()` - JSON 解析/序列化

**证据位置**: `interface/kits/jsvm.h:489-1608`

### 3. 值操作（Value Operations）

**功能**: 创建、检查、转换 JavaScript 值。

**API 数量**: ~50 个

**主要 API**:

#### 值创建
- `OH_JSVM_CreateUndefined()` / `OH_JSVM_CreateNull()` / `OH_JSVM_CreateBoolean()` 等
- `OH_JSVM_CreateInt32()` / `OH_JSVM_CreateUint32()` / `OH_JSVM_CreateInt64()` 等
- `OH_JSVM_CreateDouble()` / `OH_JSVM_CreateBigInt()` / `OH_JSVM_CreateString()` 等
- `OH_JSVM_CreateArray()` / `OH_JSVM_CreateArrayWithLength()`
- `OH_JSVM_CreateArraybuffer()` / `OH_JSVM_CreateTypedarray()` / `OH_JSVM_CreateDataview()`
- `OH_JSVM_CreateObject()` / `OH_JSVM_CreateFunction()` / `OH_JSVM_CreateSymbol()` 等

#### 值检查
- `OH_JSVM_Typeof()` - 获取值类型
- `OH_JSVM_IsUndefined()` / `OH_JSVM_IsNull()` / `OH_JSVM_IsBoolean()` 等
- `OH_JSVM_IsArray()` / `OH_JSVM_IsArrayBuffer()` / `OH_JSVM_IsTypedArray()` 等
- `OH_JSVM_IsError()` / `OH_JSVM_IsPromise()` / `OH_JSVM_IsProxy()` 等

#### 值转换
- `OH_JSVM_GetValue...()` 系列 - 从 JSVM_Value 转换为 C/C++ 类型
- `OH_JSVM_GetValueString...()` - 字符串转换
- `OH_JSVM_GetValueArray...()` - 数组转换

**证据位置**: `interface/kits/jsvm.h:907-2500+`

### 4. 属性操作（Property Operations）

**功能**: 操作 JavaScript 对象的属性。

**API 数量**: ~30 个

**主要 API**:
- `OH_JSVM_SetProperty()` / `OH_JSVM_GetProperty()` / `OH_JSVM_HasProperty()` / `OH_JSVM_DeleteProperty()`
- `OH_JSVM_SetNamedProperty()` / `OH_JSVM_GetNamedProperty()` / `OH_JSVM_HasNamedProperty()`
- `OH_JSVM_SetElement()` / `OH_JSVM_GetElement()` / `OH_JSVM_HasElement()` / `OH_JSVM_DeleteElement()`
- `OH_JSVM_GetPropertyNames()` / `OH_JSVM_GetAllPropertyNames()`
- `OH_JSVM_DefineProperties()` / `OH_JSVM_DefineClass()`

**证据位置**: `interface/kits/jsvm.h:2072-1993`（分散）

### 5. 函数操作（Function Operations）

**功能**: 创建、调用 JavaScript 函数。

**API 数量**: ~20 个

**主要 API**:
- `OH_JSVM_CreateFunction()` - 创建函数
- `OH_JSVM_CreateFunctionWithScript()` - 从脚本创建函数
- `OH_JSVM_CallFunction()` - 调用函数
- `OH_JSVM_New()` - 构造函数调用
- `OH_JSVM_GetInstanceData()` / `OH_JSVM_SetInstanceData()` - 实例数据管理

**证据位置**: `interface/kits/jsvm.h:1832-1860`

### 6. 错误处理（Error Handling）

**功能**: 抛出、捕获、检查 JavaScript 错误。

**API 数量**: ~15 个

**主要 API**:
- `OH_JSVM_Throw()` / `OH_JSVM_ThrowError()` / `OH_JSVM_ThrowTypeError()` 等
- `OH_JSVM_CreateError()` / `OH_JSVM_CreateTypeError()` / `OH_JSVM_CreateRangeError()` 等
- `OH_JSVM_GetAndClearLastException()` / `OH_JSVM_IsExceptionPending()`
- `OH_JSVM_GetLastErrorInfo()` - 获取最后错误信息

**证据位置**: `interface/kits/jsvm.h:628-752`

### 7. 引用管理（Reference Management）

**功能**: 管理 JavaScript 值的引用计数。

**API 数量**: ~8 个

**主要 API**:
- `OH_JSVM_CreateReference()` / `OH_JSVM_CreateDataReference()` - 创建引用
- `OH_JSVM_DeleteReference()` - 删除引用
- `OH_JSVM_ReferenceRef()` / `OH_JSVM_ReferenceUnref()` - 引用计数增减
- `OH_JSVM_GetReferenceValue()` / `OH_JSVM_GetReferenceData()` - 获取引用值

**证据位置**: `interface/kits/jsvm.h:823-904`

### 8. 调试与性能分析（Debugging & Profiling）

**功能**: 提供调试和性能分析能力。

**API 数量**: ~15 个

**主要 API**:
- `OH_JSVM_OpenInspector()` / `OH_JSVM_CloseInspector()` / `OH_JSVM_WaitForDebugger()` - Inspector
- `OH_JSVM_StartCpuProfiler()` / `OH_JSVM_StopCpuProfiler()` - CPU Profiler
- `OH_JSVM_TakeHeapSnapshot()` - Heap Snapshot
- `OH_JSVM_GetVMInfo()` - 获取 VM 信息
- `OH_JSVM_GetHeapStatistics()` - 获取 Heap 统计
- `OH_JSVM_MemoryPressureNotification()` - 内存压力通知

**证据位置**: `interface/kits/jsvm.h:1641-1774`

### 9. 内存管理（Memory Management）

**功能**: 监控和控制 JSVM 内存使用。

**API 数量**: ~10 个

**主要 API**:
- `OH_JSVM_GetHeapStatistics()` - 获取 Heap 统计
- `OH_JSVM_MemoryPressureNotification()` - 通知内存压力
- `OH_JSVM_GetExternalMemory()` / `OH_JSVM_AdjustExternalMemory()` - 外部内存管理
- `OH_JSVM_LowMemoryNotification()` - 低内存通知

**证据位置**: `interface/kits/jsvm.h:1650-1681`

### 10. Promise 与异步（Promise & Async）

**功能**: 管理 Promise 和异步操作。

**API 数量**: ~10 个

**主要 API**:
- `OH_JSVM_CreatePromise()` - 创建 Promise
- `OH_JSVM_ResolvePromise()` / `OH_JSVM_RejectPromise()` - 解决/拒绝 Promise
- `OH_JSVM_PromiseRejectCallback()` - Promise 拒绝回调

**证据位置**: `interface/kits/jsvm.h`（Promise 相关 API）

### 11. 数组与缓冲区（Array & Buffer）

**功能**: 创建和操作数组、ArrayBuffer、TypedArray、DataView。

**API 数量**: ~30 个

**主要 API**:
- `OH_JSVM_CreateArray()` / `OH_JSVM_CreateArrayWithLength()`
- `OH_JSVM_CreateArraybuffer()` / `OH_JSVM_GetArrayBufferInfo()` / `OH_JSVM_DetachArrayBuffer()`
- `OH_JSVM_CreateTypedarray()` / `OH_JSVM_GetTypedarrayInfo()`
- `OH_JSVM_CreateDataview()` / `OH_JSVM_GetDataviewInfo()`
- `OH_JSVM_AllocateArrayBufferBackingStoreData()` / `OH_JSVM_FreeArrayBufferBackingStoreData()`
- `OH_JSVM_CreateArrayBufferFromBackingStoreData()`

**证据位置**: `interface/kits/jsvm.h:914-996`

### 12. 对象操作（Object Operations）

**功能**: 创建和操作 JavaScript 对象。

**API 数量**: ~20 个

**主要 API**:
- `OH_JSVM_CreateObject()` - 创建对象
- `OH_JSVM_CreateSymbol()` / `OH_JSVM_SymbolFor()` - Symbol 操作
- `OH_JSVM_CreateProxy()` / `OH_JSVM_IsProxy()` / `OH_JSVM_ProxyGetTarget()` - Proxy 操作
- `OH_JSVM_GetPrototype()` / `OH_JSVM_SetPrototype()` - 原型操作
- `OH_JSVM_GetGlobalObject()` - 获取全局对象

**证据位置**: `interface/kits/jsvm.h:1039-1127`

### 13. 正则表达式（RegExp）

**功能**: 创建和操作正则表达式。

**API 数量**: ~10 个

**主要 API**:
- `OH_JSVM_CreateRegExp()` - 创建正则表达式
- `OH_JSVM_RegExpGetInfo()` - 获取正则表达式信息
- `OH_JSVM_NewInstance()` - 创建正则表达式实例

**证据位置**: `interface/kits/jsvm.h`（RegExp 相关 API）

### 14. WebAssembly

**功能**: 编译和实例化 WebAssembly 模块。

**API 数量**: ~10 个

**主要 API**:
- `OH_JSVM_CompileWasm()` - 编译 WASM
- `OH_JSVM_InstantiateWasm()` - 实例化 WASM
- `OH_JSVM_CreateWasmCache()` / `OH_JSVM_FreeWasmCache()` - WASM 缓存

**证据位置**: `interface/kits/jsvm.h`（WASM 相关 API）

---

## API 命名规范

### 前缀

所有 JSVM API 函数都以 `OH_JSVM_` 开头（OH = OpenHarmony）。

**证据位置**: `interface/kits/jsvm.h:112-311`（所有 API 声明）

### 命名模式

JSVM API 遵循以下命名模式：

#### 动词 + 名词
- `OH_JSVM_Create...()` - 创建对象
- `OH_JSVM_Get...()` - 获取值/属性
- `OH_JSVM_Set...()` - 设置值/属性
- `OH_JSVM_Delete...()` - 删除对象/属性
- `OH_JSVM_Is...()` - 检查类型/条件

#### 动词 + 宾语 + 操作
- `OH_JSVM_CreateReference()` - 创建引用
- `OH_JSVM_GetReferenceValue()` - 获取引用值
- `OH_JSVM_SetInstanceData()` - 设置实例数据

#### 特殊模式
- `OH_JSVM_Throw...()` - 抛出错误
- `OH_JSVM_New()` - 构造函数调用
- `OH_JSVM_Call...()` - 调用函数

---

## 状态码（Status Codes）

JSVM API 使用 `JSVM_Status` 枚举表示函数执行状态。

### 状态码列表

**证据位置**: `interface/kits/jsvm_types.h:277-334`

| 状态码 | 值 | 说明 |
|--------|-----|------|
| `JSVM_OK` | 0 | 成功 |
| `JSVM_INVALID_ARG` | 1 | 参数无效 |
| `JSVM_OBJECT_EXPECTED` | 2 | 期望对象类型 |
| `JSVM_STRING_EXPECTED` | 3 | 期望字符串类型 |
| `JSVM_NAME_EXPECTED` | 4 | 期望名称类型 |
| `JSVM_FUNCTION_EXPECTED` | 5 | 期望函数类型 |
| `JSVM_NUMBER_EXPECTED` | 6 | 期望数字类型 |
| `JSVM_BOOLEAN_EXPECTED` | 7 | 期望布尔类型 |
| `JSVM_ARRAY_EXPECTED` | 8 | 期望数组类型 |
| `JSVM_GENERIC_FAILURE` | 9 | 通用失败 |
| `JSVM_PENDING_EXCEPTION` | 10 | 有待处理的异常 |
| `JSVM_CANCELLED` | 11 | 操作已取消 |
| `JSVM_ESCAPE_CALLED_TWICE` | 12 | Escape 调用了两次 |
| `JSVM_HANDLE_SCOPE_MISMATCH` | 13 | Handle Scope 不匹配 |
| `JSVM_CALLBACK_SCOPE_MISMATCH` | 14 | Callback Scope 不匹配 |
| `JSVM_QUEUE_FULL` | 15 | 队列已满 |
| `JSVM_CLOSING` | 16 | 正在关闭 |
| `JSVM_BIGINT_EXPECTED` | 17 | 期望 BigInt 类型 |
| `JSVM_DATE_EXPECTED` | 18 | 期望 Date 类型 |
| `JSVM_ARRAYBUFFER_EXPECTED` | 19 | 期望 ArrayBuffer 类型 |
| `JSVM_DETACHABLE_ARRAYBUFFER_EXPECTED` | 20 | 期望可分离的 ArrayBuffer 类型 |
| `JSVM_WOULD_DEADLOCK` | 21 | 可能导致死锁 |
| `JSVM_NO_EXTERNAL_BUFFERS_ALLOWED` | 22 | 不允许外部缓冲区 |
| `JSVM_CANNOT_RUN_JS` | 23 | 无法运行 JS 代码 |
| `JSVM_INVALID_TYPE` | 24 | 无效类型（API 18+） |
| `JSVM_JIT_MODE_EXPECTED` | 25 | 期望 JIT 模式（API 18+） |

---

## API 版本管理

### JSVM_VERSION

JSVM 使用 `JSVM_VERSION` 宏控制 API 版本兼容性。

**当前版本**: 8

**定义位置**: `interface/kits/jsvm.h:51-63`

### 版本特性

JSVM API 从版本 11 开始支持，版本 18 增加了一些新特性。

**证据位置**:
- `interface/kits/jsvm.h:29` - @since 11
- `interface/kits/jsvm.h:110` - @since 18

---

## 线程模型

### 线程安全

JSVM API **不是线程安全**的，必须在创建 Environment 的线程中调用 API。

**建议**:
- 每个线程使用独立的 Environment
- 避免跨线程共享 JSVM_Value
- 使用引用计数跨线程传递 JSVM_Value

### 异步操作

JSVM API 提供了一些异步操作支持：

- Promise
- Microtask
- Worker（TODO: 需确认）

**证据位置**:
- `interface/kits/jsvm_types.h:741-748` - JSVM_MicrotaskPolicy
- `interface/kits/jsvm.h:344` - `OH_JSVM_SetMicrotaskPolicy()`

---

## 相关链接

- [对外 API 详细文档](./05_Public_API_Details.md) - API 清单与使用指南
- [概览](./01_Overview.md) - 了解项目定位
- [架构说明](./04_Architecture.md) - 理解 API 调用链
