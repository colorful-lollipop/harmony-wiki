# N-API 接口参考

本文档记录 cangjie_ark_interop 提供的所有公开 N-API 接口。

## 命名空间

| 命名空间 | 模块 | 说明 |
|----------|------|------|
| `ohos.ark_interop` | ark_interop | 核心互操作 API |
| `ohos.ark_interop_helper` | ark_interop_helper | 互操作工具函数 |
| `ohos.business_exception` | business_exception | 异常类定义 |
| `ohos.ffi` | ffi | C 语言互操作 |
| `kit.CangjieKit` | CangjieKit | Kit 接口导出 |

---

## JSRuntime

**位置**: `ohos/ark_interop/js_runtime.cj:68`

**命名空间**: `ohos.ark_interop`

**说明**: 表示 ArkTS 运行时实例，每个 ArkTS 运行时对应一个 JSRuntime 实例。

### 构造函数

| 方法 | 说明 | 抛出异常 |
|------|------|----------|
| `init()` | 创建新的 JSRuntime 实例 | BusinessException (34300014) |

**示例**:
```cangjie
let runtime = JSRuntime()
let context = runtime.mainContext
```

### 属性

| 属性 | 类型 | 说明 | 抛出异常 |
|------|------|------|----------|
| `mainContext` | JSContext | 获取关联的执行上下文 | - |

### 方法

| 方法 | 返回类型 | 说明 |
|------|----------|------|
| `getNapiEnv()` | CPointer\<Unit\> | 获取 napi 环境指针 |

### 错误码

| 错误码 | 说明 |
|--------|------|
| 34300014 | Create ArkTS engine fail |
| 34300002 | Outside error occurred |

---

## JSContext

**位置**: `ohos/ark_interop/jscontext.cj`

**命名空间**: `ohos.ark_interop`

**说明**: 表示与 JSRuntime 关联的执行上下文，保存全局对象，提供模块加载、JSValue 访问等方法。

### 类型别名

| 别名 | 原始类型 | 说明 |
|------|----------|------|
| `JSEnv` | IntNative | ArkTS 环境句柄 |
| `napi_env` | CPointer\<Unit\> | napi 环境指针 |
| `napi_value` | CPointer\<Unit\> | napi 值指针 |

### 核心方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `getOrCreate(env, tidGetter)` | JSEnv, ()->UInt64 | JSContext | 获取或创建上下文 |
| `function(fn)` | (JSContext, JSCallInfo) -> T | JSFunction | 创建 JS 函数 |
| `postJSTask(task)` | ()->Unit | Unit | 提交 JS 任务 |
| `checkLifecycleAndThread()` | - | Unit | 检查生命周期和线程 |

---

## JSCallInfo

**位置**: `ohos/ark_interop/js_func.cj:54`

**命名空间**: `ohos.ark_interop`

**说明**: 表示 ArkTS 调用仓颉方法时的调用信息，用于获取参数数量和参数列表。

### 属性

| 属性 | 类型 | 说明 | 抛出异常 |
|------|------|------|----------|
| `count` | Int64 | 输入参数数量 | BusinessException (34300003, 34300004) |
| `thisArg` | JSValue | this 指针 | BusinessException (34300003, 34300004) |

### 方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `get(index)` | UInt32 | JSValue | 获取指定索引的参数 |

### 错误码

| 错误码 | 说明 |
|--------|------|
| 34300003 | Accessing reference is beyond reach |
| 34300004 | Thread mismatch |

---

## JSObject

**位置**: `ohos/ark_interop/jsobject.cj:89`

**命名空间**: `ohos.ark_interop`

**说明**: ArkTS 对象的安全引用类型。

### 静态方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `create(context)` | JSContext | JSObject | 创建新对象 |
| `create(context, value)` | JSContext, JSValue_ | JSObject | 从现有值创建 |

### 实例方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `getProperty(key)` | JSValue | JSValue | 获取属性值 |
| `setProperty(key, value)` | JSValue, JSValue | Unit | 设置属性值 |
| `hasProperty(key)` | JSValue | Bool | 检查属性存在 |
| `deleteProperty(key)` | JSValue | Bool | 删除属性 |
| `callMethod(name, args)` | String, Array\<JSValue\> | JSValue | 调用方法 |

---

## JSArray

**位置**: `ohos/ark_interop/jsarray.cj:42`

**命名空间**: `ohos.ark_interop`

**说明**: ArkTS 数组的安全引用类型，支持长度获取、元素读写。

### 构造函数

| 构造方法 | 参数 | 说明 |
|----------|------|------|
| `init(context, value)` | JSContext, JSValue_ | 从现有值创建 |
| `init(context, arr)` | JSContext, Array\<JSValue\> | 从 Cangjie 数组创建 |

### 属性

| 属性 | 类型 | 说明 | 抛出异常 |
|------|------|------|----------|
| `length` | UInt32 | 数组长度 | BusinessException (34300003, 34300004) |

### 方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `get(index)` | UInt32 | JSValue | 获取元素 |
| `set(index, value)` | UInt32, JSValue | Unit | 设置元素 |
| `toArray()` | - | Array\<JSValue\> | 转换为 Cangjie 数组 |

---

## JSValue 类型系列

**命名空间**: `ohos.ark_interop`

### JSNull / JSUndefined

```cangjie
public let jsNull: JSValue    // 表示 null
public let jsUndefined: JSValue  // 表示 undefined
```

### JSString

**位置**: `ohos/ark_interop/jsstring.cj`

```cangjie
public class JSString <: JSHeapObject {
    public prop length: UInt32  // 字符串长度
    public func toString(): String  // 转换为 Cangjie String
}
```

### 类型转换接口

| 接口 | 方法 | 说明 |
|------|------|------|
| `ToJSValue` | `toJSValue(context: JSContext): JSValue` | 转换为 JSValue |
| `ToJSValue` | `fromJSValue(context: JSContext, value: JSValue): Self` | 从 JSValue 转换 |

---

## BusinessException

**位置**: `ohos/business_exception/business_exception.cj:47`

**命名空间**: `ohos.business_exception`

**说明**: 通用业务异常类，继承自 Exception。

### 属性

| 属性 | 类型 | 说明 |
|------|------|------|
| `code` | Int32 | 错误码 |
| `message` | String | 错误消息 |

### 方法

| 方法 | 参数 | 返回值 | 说明 |
|------|------|--------|------|
| `getData<T>()` | - | ?T | 获取附加业务数据 |
| `toString()` | - | String | 转换为字符串 |

### 错误码定义

| 错误码 | 常量名 | 说明 |
|--------|--------|------|
| 201 | ERR_NO_PERMISSION | 权限验证失败 |
| 202 | ERR_NOT_SYSTEM_APP | 非系统应用调用系统 API |
| 401 | ERR_PARAMETER_ERROR | 参数错误 |
| 801 | ERR_NOT_SUPPOERTED | 能力不支持 |

### 互操作相关错误码

| 错误码 | 说明 |
|--------|------|
| 34300001 | 数组索引越界 |
| 34300002 | 外部错误 |
| 34300003 | 引用访问越界 |
| 34300004 | 线程不匹配 |
| 34300005 | 类型不匹配 |
| 34300014 | 创建 ArkTS 引擎失败 |

---

## AsyncCallback

**位置**: `ohos/business_exception/business_exception.cj:105`

**类型定义**:

```cangjie
public type AsyncCallback<T> = (Option<BusinessException>, Option<T>) -> Unit
```

**参数说明**:
- `Option<BusinessException>`: 错误信息 (None 表示成功)
- `Option<T>`: 成功时的返回值 (None 表示无返回值)

---

## 模块加载

**位置**: `ohos/ark_interop/js_module.cj`

```cangjie
public class JSModule {
    // 加载 ArkTS 模块
    public static func import(context: JSContext, entryPoint: String): JSModule

    // 获取模块导出的对象
    public prop exports: JSObject
}
```

---

## 同步/异步模式

### 同步调用

```cangjie
let result = jsContext.eval("1 + 1")  // 同步执行
```

### 异步调用 (Promise)

```cangjie
let promise = jsContext.evalAsync("fetch(url)")
promise.then({ ctx, info =>
    // 处理结果
})
```

### 异步调用 (Callback)

```cangjie
let callback: AsyncCallback<Int> = { error, result =>
    match (error) {
        case Some(e) => print("Error: ${e.message}")
        case None => print("Result: ${result.get()}")
    }
}
context.callAsync(target, method, args, callback)
```

---

## 参数校验

### 线程安全

```cangjie
// 所有 JSValue 操作必须在 ArkTS 线程执行
context.checkLifecycleAndThread()  // 抛出 34300004 如果线程不匹配
```

### 类型检查

```cangjie
// 自动类型检查，类型不匹配抛出 34300005
let str: JSString = value.asString()
```

### 边界检查

```cangjie
// 数组索引越界抛出 34300001
let element = array.get(index)
```
