# N-API 接口参考

## 模块注册

### 标准模块注册

**证据**：`native_node_api.h:35-44`

```cpp
typedef struct napi_module_with_js {
    int nm_version = 0;
    unsigned int nm_flags = 0;
    const char* nm_filename = nullptr;
    napi_addon_register_func nm_register_func = nullptr;
    const char* nm_modname = nullptr;
    void* nm_priv = nullptr;
    NAPIGetJSCode nm_get_abc_code = nullptr;
    NAPIGetJSCode nm_get_js_code = nullptr;
} napi_module_with_js;
```

**注册宏和函数**：

| API | 描述 | 证据位置 |
|-----|------|----------|
| `napi_module_register()` | 注册模块 | `native_node_api.h:58` |
| `napi_module_with_js_register()` | 注册带 JS 代码的模块 | `native_node_api.h:58` |

**示例**（从 README）：

```cpp
#include "napi/native_api.h"
#include "napi/native_node_api.h"

static napi_value AppExport(napi_env env, napi_value exports)
{
    static napi_property_descriptor desc[] = {
        DECLARE_NAPI_FUNCTION("getAppName", JSGetAppName),
    };
    NAPI_CALL(env, napi_define_properties(env, exports, sizeof(desc) / sizeof(desc[0]), desc));
    return exports;
}

static napi_module appModule = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = AppExport,
    .nm_modname = "app",
    .nm_priv = ((void*)0),
    .reserved = {0}
};

extern "C" __attribute__((constructor)) void AppRegister()
{
    napi_module_register(&appModule);
}
```

## 错误处理宏

**证据**：`native_common.h:21-56`

| 宏 | 描述 | 示例 |
|----|------|------|
| `NAPI_CALL(env, theCall)` | 检查状态，失败则抛错并返回 | `NAPI_CALL(env, napi_create_string_utf8(...))` |
| `NAPI_CALL_RETURN_VOID(env, theCall)` | Void 版本 | `NAPI_CALL_RETURN_VOID(env, napi_set_property(...))` |
| `NAPI_ASSERT(env, assertion, message)` | 断言，失败抛错 | `NAPI_ASSERT(env, argc > 0, "Need arguments")` |
| `GET_AND_THROW_LAST_ERROR(env)` | 获取并抛出最后错误 | `GET_AND_THROW_LAST_ERROR(env)` |

## 属性声明宏

**证据**：`native_common.h:59-167`

| 宏 | 描述 |
|----|------|
| `DECLARE_NAPI_FUNCTION(name, func)` | 声明导出函数 |
| `DECLARE_NAPI_FUNCTION_WITH_DATA(name, func, data)` | 带数据的函数 |
| `DECLARE_NAPI_PROPERTY(name, val)` | 声明属性 |
| `DECLARE_NAPI_STATIC_FUNCTION(name, func)` | 声明静态函数 |
| `DECLARE_NAPI_STATIC_PROPERTY(name, val)` | 声明静态属性 |
| `DECLARE_NAPI_GETTER(name, getter)` | 声明 getter |
| `DECLARE_NAPI_SETTER(name, setter)` | 声明 setter |
| `DECLARE_NAPI_GETTER_SETTER(name, getter, setter)` | 声明 getter/setter 对 |

## 类型转换

### 创建 NAPI 值

| API | 描述 | 证据位置 |
|-----|------|----------|
| `napi_create_string_utf8()` | UTF-8 字符串 | `native_api.h` |
| `napi_create_string_utf16()` | UTF-16 字符串 | `native_api.h:46-49` |
| `napi_create_int32()` | 32 位整数 | 标准 N-API |
| `napi_create_int64()` | 64 位整数 | 标准 N-API |
| `napi_create_double()` | 双精度浮点 | 标准 N-API |
| `napi_create_object()` | JS 对象 | 标准 N-API |
| `napi_create_array()` | JS 数组 | 标准 N-API |
| `napi_create_array_with_length(n)` | 指定长度数组 | 标准 N-API |
| `napi_create_arraybuffer()` | ArrayBuffer | 标准 N-API |
| `napi_create_typedarray()` | TypedArray | 标准 N-API |
| `napi_create_dataview()` | DataView | 标准 N-API |
| `napi_create_buffer()` | Buffer | 标准 N-API |
| `napi_create_date()` | Date | 标准 N-API |
| `napi_create_symbol()` | Symbol | 标准 N-API |
| `napi_create_error()` / `napi_create_type_error()` / `napi_create_range_error()` | 错误对象 | 标准 N-API |
| `napi_create_bigint_int64()` / `napi_create_bigint_uint64()` | 大整数 | 标准 N-API |
| `napi_create_promise()` | Promise | 标准 N-API |

### 获取 NAPI 值

| API | 描述 | 证据位置 |
|-----|------|----------|
| `napi_get_value_string_utf8()` | 获取 UTF-8 字符串 | 标准 N-API |
| `napi_get_value_string_utf16()` | 获取 UTF-16 字符串 | `native_api.h:51-55` |
| `napi_get_value_int32()` | 获取 32 位整数 | 标准 N-API |
| `napi_get_value_int64()` | 获取 64 位整数 | 标准 N-API |
| `napi_get_value_double()` | 获取双精度浮点 | 标准 N-API |
| `napi_get_value_bool()` | 获取布尔值 | 标准 N-API |
| `napi_get_value_external()` | 获取外部数据 | 标准 N-API |
| `napi_get_array_length()` | 获取数组长度 | 标准 N-API |

### 类型检查

| API | 描述 | 证据位置 |
|-----|------|----------|
| `napi_typeof()` | 获取值类型 | 标准 N-API |
| `napi_is_array()` | 是否为数组 | 标准 N-API |
| `napi_is_arraybuffer()` | 是否为 ArrayBuffer | 标准 N-API |
| `napi_is_typedarray()` | 是否为 TypedArray | 标准 N-API |
| `napi_is_dataview()` | 是否为 DataView | 标准 N-API |
| `napi_is_buffer()` | 是否为 Buffer | 标准 N-API |
| `napi_is_date()` | 是否为 Date | 标准 N-API |
| `napi_is_error()` | 是否为错误 | 标准 N-API |
| `napi_is_promise()` | 是否为 Promise | 标准 N-API |
| `napi_is_callable()` | 是否可调用 | `native_node_api.h:59` |

## 对象操作

### 属性操作

| API | 描述 |
|-----|------|
| `napi_set_property()` | 设置属性 |
| `napi_get_property()` | 获取属性 |
| `napi_has_property()` | 检查属性存在 |
| `napi_delete_property()` | 删除属性 |
| `napi_set_named_property()` | 设置命名属性 |
| `napi_get_named_property()` | 获取命名属性 |
| `napi_has_named_property()` | 检查命名属性 |
| `napi_define_properties()` | 批量定义属性 |

### 元素操作

| API | 描述 |
|-----|------|
| `napi_set_element()` | 设置元素 |
| `napi_get_element()` | 获取元素 |
| `napi_has_element()` | 检查元素存在 |
| `napi_delete_element()` | 删除元素 |

## 异步操作

### Async Work

| API | 描述 | 证据位置 |
|-----|------|----------|
| `napi_create_async_work()` | 创建异步工作 | 标准 N-API |
| `napi_queue_async_work()` | 队列异步工作 | 标准 N-API |
| `napi_queue_async_work_with_qos()` | 带 QoS 的队列 | `native_api.h:81` |
| `napi_delete_async_work()` | 删除异步工作 | 标准 N-API |

### Promise

| API | 描述 |
|-----|------|
| `napi_create_promise()` | 创建 Promise |
| `napi_resolve_deferred()` | Resolve Promise |
| `napi_reject_deferred()` | Reject Promise |

### Thread-safe Function (TSFN)

| API | 描述 |
|-----|------|
| `napi_create_threadsafe_function()` | 创建 TSFN |
| `napi_call_threadsafe_function()` | 调用 TSFN |
| `napi_call_threadsafe_function_with_priority()` | 带优先级调用 |
| `napi_acquire_threadsafe_function()` | 获取 TSFN |
| `napi_release_threadsafe_function()` | 释放 TSFN |
| `napi_ref_threadsafe_function()` | 引用 TSFN |
| `napi_unref_threadsafe_function()` | 取消引用 |

## 错误处理

| API | 描述 | 证据位置 |
|-----|------|----------|
| `napi_throw()` | 抛出 JS 值 | 标准 N-API |
| `napi_throw_error()` | 抛出错误 | 标准 N-API |
| `napi_throw_type_error()` | 抛出 TypeError | 标准 N-API |
| `napi_throw_range_error()` | 抛出 RangeError | 标准 N-API |
| `napi_throw_business_error()` | 抛出业务错误 | `native_api.h:158` |
| `napi_is_error()` | 检查是否为错误 | 标准 N-API |
| `napi_get_and_clear_last_exception()` | 获取并清除异常 | 标准 N-API |
| `napi_is_exception_pending()` | 检查异常待处理 | 标准 N-API |
| `napi_fatal_exception()` | 致命异常 | `native_api.h:44` |
| `napi_fatal_error()` | 致命错误 | 标准 N-API |

## 对象生命周期管理

### Wrap/Unwrap

| API | 描述 | 证据位置 |
|-----|------|----------|
| `napi_wrap()` | 包装 native 数据到 JS 对象 | 标准 N-API |
| `napi_unwrap()` | 解包获取 native 数据 | 标准 N-API |
| `napi_wrap_with_size()` | 带大小提示的包装 | `native_api.h:81-87` |
| `napi_wrap_async_finalizer()` | 带异步 finalizer 的包装 | `native_api.h:88-94` |
| `napi_wrap_s()` / `napi_unwrap_s()` | 类型安全包装 | `native_node_api.h:101-111` |
| `napi_remove_wrap()` | 移除包装 | 标准 N-API |
| `napi_create_external()` | 创建外部对象 | 标准 N-API |
| `napi_create_external_with_size()` | 带大小提示的外部对象 | `native_api.h:95-100` |
| `napi_get_value_external()` | 获取外部数据 | 标准 N-API |
| `napi_wrap_enhance()` | 增强包装 | `native_api.h:150-157` |

### Sendable 对象（多线程）

| API | 描述 | 证据位置 |
|-----|------|----------|
| `napi_create_sendable_object_with_properties()` | 创建可发送对象 | `native_api.h:169-172` |
| `napi_create_sendable_array()` | 创建可发送数组 | `native_api.h:173` |
| `napi_create_sendable_arraybuffer()` | 创建可发送 ArrayBuffer | `native_api.h:175-176` |
| `napi_create_sendable_typedarray()` | 创建可发送 TypedArray | `native_api.h:177-182` |
| `napi_is_sendable()` | 检查是否可发送 | `native_api.h:187` |
| `napi_wrap_sendable()` / `napi_unwrap_sendable()` | 可发送对象包装 | `native_api.h:188-199` |

### Reference

| API | 描述 |
|-----|------|
| `napi_create_reference()` | 创建引用 |
| `napi_delete_reference()` | 删除引用 |
| `napi_reference_ref()` | 增加引用计数 |
| `napi_reference_unref()` | 减少引用计数 |
| `napi_get_reference_value()` | 获取引用值 |
| `napi_create_strong_reference()` | 创建强引用 |
| `napi_delete_strong_reference()` | 删除强引用 |

### Scope

| API | 描述 |
|-----|------|
| `napi_open_handle_scope()` | 打开作用域 |
| `napi_close_handle_scope()` | 关闭作用域 |
| `napi_open_escapable_handle_scope()` | 打开可逃逸作用域 |
| `napi_close_escapable_handle_scope()` | 关闭可逃逸作用域 |
| `napi_escape_handle()` | 逃逸到外层作用域 |

## 运行时管理

| API | 描述 | 证据位置 |
|-----|------|----------|
| `napi_create_ark_runtime()` | 创建 Ark 运行时 | `native_api.h:113` |
| `napi_destroy_ark_runtime()` | 销毁 Ark 运行时 | `native_api.h:114` |
| `napi_create_runtime()` | 创建通用运行时 | `native_node_api.h:60` |
| `napi_destroy_runtime()` | 销毁通用运行时 | `native_node_api.h:61` |

## 模块加载

| API | 描述 | 证据位置 |
|-----|------|----------|
| `napi_load_module()` | 加载 ES 模块 | `native_api.h:87` |
| `napi_load_module_with_info()` | 带信息加载模块 | `native_api.h:88-91` |

## 序列化

| API | 描述 | 证据位置 |
|-----|------|----------|
| `napi_serialize()` | 序列化对象 | `native_api.h:116-120` |
| `napi_deserialize()` | 反序列化 | `native_api.h:121` |
| `napi_delete_serialization_data()` | 删除序列化数据 | `native_api.h:122` |

## ArkTS 互操作 API

**证据**：`ark_interop_napi.h`

### 值创建

| API | 描述 |
|-----|------|
| `ARKTS_CreateObject()` | 创建对象 |
| `ARKTS_CreateArray()` | 创建数组 |
| `ARKTS_CreateUtf8()` | 创建 UTF-8 字符串 |
| `ARKTS_CreateF64()` | 创建浮点数 |
| `ARKTS_CreateI32()` | 创建整数 |

### 属性操作

| API | 描述 |
|-----|------|
| `ARKTS_GetProperty()` | 获取属性 |
| `ARKTS_SetProperty()` | 设置属性 |
| `ARKTS_DefineOwnProperty()` | 定义属性 |

### 函数调用

| API | 描述 |
|-----|------|
| `ARKTS_Call()` | 调用函数 |
| `ARKTS_New()` | 创建对象 |

### Promise

| API | 描述 |
|-----|------|
| `ARKTS_CreatePromiseCapability()` | 创建 Promise Capability |
| `ARKTS_PromiseCapabilityResolve()` | Resolve |
| `ARKTS_PromiseCapabilityReject()` | Reject |

### 引擎操作

| API | 描述 |
|-----|------|
| `ARKTS_CreateEngine()` | 创建引擎 |
| `ARKTS_DestroyEngine()` | 销毁引擎 |
| `ARKTS_GetContext()` | 获取上下文 |
| `ARKTS_Require()` | 导入模块 |

---

**相关文档**：
- [架构说明](./03_Architecture.md)
- [构建系统](./05_Build_System.md)
- [示例代码](../sample/)
