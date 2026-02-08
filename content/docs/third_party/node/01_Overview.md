# 原始库简介

## 一、Node.js N-API 概述

### 1.1 什么是 N-API

N-API 是 Node.js 官方提供的 C/C++ 接口标准，用于编写与 JavaScript 运行时交互的原生模块（Native Addon）。其设计目标是在 Node.js 不同版本之间保持 ABI（应用程序二进制接口）稳定性，使得原生模块无需在不同 Node.js 版本之间重新编译即可运行。

N-API 的核心设计理念是「一次编写，到处运行」。通过将接口层与底层 JavaScript 引擎解耦，N-API 使得原生模块可以兼容 V8 引擎的不同版本，甚至可以在非 V8 引擎（如 Chakra、Ark）上运行。

### 1.2 N-API 的技术特性

**ABI 稳定性**：N-API 被设计为 ABI 稳定的接口层。自 Node.js 8.0.0 起，N-API 进入稳定状态，承诺接口的二进制兼容性。这意味着使用 N-API 编写的原生模块可以在不同版本的 Node.js 上运行，无需重新编译。

**引擎无关性**：N-API 抽象了 JavaScript 引擎的实现细节，原生模块开发者无需直接操作 V8 的 C++ API。这大大降低了模块维护的复杂度，同时也使得 N-API 可以运行在任何实现了 N-API 规范的 JavaScript 引擎上。

**版本演进**：N-API 采用版本化演进机制，不同版本支持不同的特性集。当前库使用的 N-API 版本为 8，引入了对象冻结（object freeze）、密封（seal）、类型标签（type tagging）等高级特性。

### 1.3 N-API 在 Node.js 生态中的位置

```
┌─────────────────────────────────────────────────────┐
│              用户 JavaScript 代码                    │
├─────────────────────────────────────────────────────┤
│                   V8 引擎                           │
│          （JavaScript 执行环境）                     │
├─────────────────────────────────────────────────────┤
│                  libuv                              │
│           （异步 I/O 事件循环）                      │
├─────────────────────────────────────────────────────┤
│                  N-API 层                            │
│         （Native Addon 接口抽象层）                  │
├─────────────────────────────────────────────────────┤
│            原生模块（C/C++ Addon）                   │
└─────────────────────────────────────────────────────┘
```

N-API 位于原生模块和底层运行时之间，向上提供统一的 C API，向下对接具体的 JavaScript 引擎实现。

## 二、头文件结构说明

### 2.1 文件清单

本库包含四个核心头文件：

| 文件名 | 功能描述 | 依赖关系 |
|--------|----------|----------|
| js_native_api_types.h | N-API 类型定义（ opaque 指针、枚举、常量） | 无 |
| js_native_api.h | N-API 核心接口函数声明 | js_native_api_types.h |
| node_api_types.h | Node.js 特有类型定义 | js_native_api_types.h |
| node_api.h | Node.js 特有接口声明 | js_native_api.h、node_api_types.h |

### 2.2 js_native_api_types.h 详解

该文件定义了 N-API 的基础类型系统，是整个接口规范的基石。

**核心类型定义**：

```c
// 环境句柄
typedef struct napi_env__* napi_env;

// JS 值句柄
typedef struct napi_value__* napi_value;

// 引用类型
typedef struct napi_ref__* napi_ref;

// 回调信息
typedef struct napi_callback_info__* napi_callback_info;

// 延迟对象（用于 Promise）
typedef struct napi_deferred__* napi_deferred;
```

这些类型均使用不透明指针（opaque pointer）定义，隐藏了内部实现细节，确保 ABI 稳定性。

**枚举类型**：

```c
// JavaScript 值类型
typedef enum {
  napi_undefined,
  napi_null,
  napi_boolean,
  napi_number,
  napi_string,
  napi_symbol,
  napi_object,
  napi_function,
  napi_external,
  napi_bigint,
} napi_valuetype;

// Typed Array 类型
typedef enum {
  napi_int8_array,
  napi_uint8_array,
  napi_uint8_clamped_array,
  napi_int16_array,
  napi_uint16_array,
  napi_int32_array,
  napi_uint32_array,
  napi_float32_array,
  napi_float64_array,
  napi_bigint64_array,
  napi_biguint64_array,
} napi_typedarray_type;

// API 调用状态
typedef enum {
  napi_ok,
  napi_invalid_arg,
  napi_object_expected,
  // ... 其他状态码
  // OpenHarmony 特有扩展
  napi_create_ark_runtime_too_many_envs = 22,
  napi_create_ark_runtime_only_one_env_per_thread = 23,
  napi_destroy_ark_runtime_env_not_exist = 24
} napi_status;
```

### 2.3 js_native_api.h 详解

该文件声明了 N-API 的核心接口函数，涵盖了 JavaScript 值的创建、获取、转换等基本操作。

**原始值创建**：

```c
// 创建基本类型和对象
napi_status napi_create_object(napi_env env, napi_value* result);
napi_status napi_create_array(napi_env env, napi_value* result);
napi_status napi_create_string_utf8(napi_env env, const char* str, size_t length, napi_value* result);
napi_status napi_create_function(napi_env env, const char* utf8name, size_t length,
                                 napi_callback cb, void* data, napi_value* result);
napi_status napi_create_error(napi_env env, napi_value code, napi_value msg, napi_value* result);
```

**值获取与转换**：

```c
// 从 JS 值获取原生数据
napi_status napi_get_value_double(napi_env env, napi_value value, double* result);
napi_status napi_get_value_string_utf8(napi_env env, napi_value value, char* buf, size_t bufsize, size_t* result);
napi_status napi_get_value_int32(napi_env env, napi_value value, int32_t* result);
```

**对象操作**：

```c
// 属性操作
napi_status napi_set_property(napi_env env, napi_value object, napi_value key, napi_value value);
napi_status napi_get_property(napi_env env, napi_value object, napi_value key, napi_value* result);
napi_status napi_has_property(napi_env env, napi_value object, napi_value key, bool* result);

// 批量定义属性
napi_status napi_define_properties(napi_env env, napi_value object,
                                   size_t property_count,
                                   const napi_property_descriptor* properties);
```

**数组操作**：

```c
// 数组检查和长度获取
napi_status napi_is_array(napi_env env, napi_value value, bool* result);
napi_status napi_get_array_length(napi_env env, napi_value value, uint32_t* result);
```

**回调和函数**：

```c
// 获取回调信息
napi_status napi_get_cb_info(napi_env env, napi_callback_info cbinfo,
                             size_t* argc, napi_value* argv,
                             napi_value* this_arg, void** data);

// 调用 JS 函数
napi_status napi_call_function(napi_env env, napi_value recv, napi_value func,
                               size_t argc, const napi_value* argv,
                               napi_value* result);

// 定义 JS 类
napi_status napi_define_class(napi_env env, const char* utf8name, size_t length,
                               napi_callback constructor, void* data,
                               size_t property_count,
                               const napi_property_descriptor* properties,
                               napi_value* result);
```

**Promise 支持**：

```c
// Promise 操作
napi_status napi_create_promise(napi_env env, napi_deferred* deferred, napi_value* promise);
napi_status napi_resolve_deferred(napi_env env, napi_deferred deferred, napi_value resolution);
napi_status napi_reject_deferred(napi_env env, napi_deferred deferred, napi_value rejection);
napi_status napi_is_promise(napi_env env, napi_value value, bool* is_promise);
```

**内存管理**：

```c
// 引用计数
napi_status napi_create_reference(napi_env env, napi_value value, uint32_t initial_refcount, napi_ref* result);
napi_status napi_reference_ref(napi_env env, napi_ref ref, uint32_t* result);
napi_status napi_reference_unref(napi_env env, napi_ref ref, uint32_t* result);

// 句柄作用域
napi_status napi_open_handle_scope(napi_env env, napi_handle_scope* result);
napi_status napi_close_handle_scope(napi_env env, napi_handle_scope scope);
```

### 2.4 node_api.h 详解

该文件声明了 Node.js 特有的 N-API 扩展接口，提供了异步操作、Buffer 等 Node.js 特有功能。

**异步操作**：

```c
// 异步工作
napi_status napi_create_async_work(napi_env env, napi_value async_resource,
                                   napi_value async_resource_name,
                                   napi_async_execute_callback execute,
                                   napi_async_complete_callback complete,
                                   void* data, napi_async_work* result);
napi_status napi_queue_async_work(napi_env env, napi_async_work work);
napi_status napi_delete_async_work(napi_env env, napi_async_work work);
```

**线程安全函数**（N-API v4+）：

```c
// 线程安全函数
napi_status napi_create_threadsafe_function(napi_env env, napi_value func,
                                            napi_value async_resource,
                                            napi_value async_resource_name,
                                            size_t max_queue_size,
                                            size_t initial_thread_count,
                                            void* thread_finalize_data,
                                            napi_finalize thread_finalize_cb,
                                            void* context,
                                            napi_threadsafe_function_call_js call_js_cb,
                                            napi_threadsafe_function* result);

napi_status napi_call_threadsafe_function(napi_threadsafe_function func,
                                           void* data,
                                           napi_threadsafe_function_call_mode is_blocking);
```

**Buffer 操作**：

```c
// Buffer 创建和管理
napi_status napi_create_buffer(napi_env env, size_t length, void** data, napi_value* result);
napi_status napi_create_buffer_copy(napi_env env, size_t length, const void* data,
                                    void** result_data, napi_value* result);
napi_status napi_get_buffer_info(napi_env env, napi_value value, void** data, size_t* length);
```

### 2.5 版本条件编译

N-API 使用条件编译来支持不同版本的接口：

```c
#if NAPI_VERSION >= 5
// N-API v5 新增的 Date 相关接口
napi_status napi_create_date(napi_env env, double time, napi_value* result);
napi_status napi_is_date(napi_env env, napi_value value, bool* is_date);
#endif

#if NAPI_VERSION >= 6
// N-API v6 新增的 BigInt 和实例数据接口
napi_status napi_create_bigint_int64(napi_env env, int64_t value, napi_value* result);
napi_status napi_set_instance_data(napi_env env, void* data, napi_finalize finalize_cb, void* finalize_hint);
#endif

#if NAPI_VERSION >= 8
// N-API v8 新增的 Object freeze/seal 和类型标签接口
napi_status napi_type_tag_object(napi_env env, napi_value value, const napi_type_tag* type_tag);
napi_status napi_object_freeze(napi_env env, napi_value object);
napi_status napi_object_seal(napi_env env, napi_value object);
#endif
```

默认的 NAPI_VERSION 定义为 8，这意味着 OpenHarmony 使用的是支持所有稳定特性的完整 N-API 接口集。

## 三、在 OpenHarmony 中的定位

### 3.1 技术选型依据

OpenHarmony 选择 Node.js N-API 作为 Native 互操作标准，主要基于以下考量：

**生态兼容性**：N-API 拥有成熟的开发者生态和丰富的第三方模块库。通过采用 N-API 标准，OpenHarmony 可以复用大量现有的 Node.js 原生模块资源。

**技术成熟度**：N-API 经过多年发展，已在 Node.js 生态中验证了其稳定性和实用性。其 ABI 稳定性承诺使得接口升级成本可控。

**引擎无关性**：OpenHarmony 自研的 Ark 引擎无需实现 V8 API，只需实现 N-API 接口规范即可支持原生模块。这种设计降低了引擎与上层应用的耦合度。

### 3.2 架构位置

在 OpenHarmony 系统架构中，N-API 位于以下位置：

```
┌─────────────────────────────────────────────────────────────┐
│                    用户应用层                               │
│              ArkTS / JavaScript 应用                        │
├─────────────────────────────────────────────────────────────┤
│                    Ark 运行时                               │
│              （JavaScript/TypeScript 执行引擎）               │
├─────────────────────────────────────────────────────────────┤
│              N-API 接口层（third_party/node）                 │
│        （C/C++ 与 JS 互操作的标准接口规范）                  │
├─────────────────────────────────────────────────────────────┤
│                 Native 模块层                               │
│          （传感器、文件系统、网络等 C/C++ 实现）              │
├─────────────────────────────────────────────────────────────┤
│                    系统内核                                 │
└─────────────────────────────────────────────────────────────┘
```

N-API 接口层作为桥梁，连接了上层的 Ark 运行时和下层的 Native 模块，使得 Native 模块可以安全、规范化地与 JavaScript/ArkTS 代码交互。

### 3.3 与其他方案对比

| 特性 | N-API | 直接使用 V8 API | 自定义接口 |
|------|-------|-----------------|------------|
| ABI 稳定性 | 良好 | 差（随 V8 版本变化） | 需自行维护 |
| 跨引擎支持 | 支持 | 仅 V8 | 仅特定引擎 |
| 学习成本 | 中等 | 高 | 中等 |
| 生态复用 | 高 | 低 | 无 |
| OpenHarmony 采用 | 是 | 否 | 否 |

N-API 在稳定性、兼容性和生态支持方面具有明显优势，是 OpenHarmony 的最佳选择。
