# N-API 参考

本文档描述 ArkCompiler ETS Runtime 的 N-API 接口规范，包括 API 清单、参数校验、错误码和绑定位置。

## N-API 概述

N-API 是 ArkCompiler ETS Runtime 对外暴露的 C++ 原生编程接口，提供稳定的二进制兼容性，允许原生模块与运行时交互而不受内部实现变化影响。

### 设计原则

- **稳定性**：API 签名长期稳定，不随内部实现变化
- **语言无关**：支持 C 和 C++ 调用
- **类型安全**：明确的类型映射和参数校验
- **错误处理**：统一的错误码机制

### 头文件位置

- **主头文件**：`ecmascript/napi/include/jsnapi.h`
- **导出头文件**：`ecmascript/napi/include/jsnapi_expo.h`
- **DFX 头文件**：`ecmascript/napi/include/dfx_jsnapi.h`

## 核心 API 分类

### 1. 运行时生命周期

| C++ API | JS API | 功能 | 绑定文件 |
|---------|--------|------|----------|
| `napi_create_runtime` | N/A | 创建 Runtime 实例 | `jsnapi.cpp:100+` |
| `napi_destroy_runtime` | N/A | 销毁 Runtime | `jsnapi.cpp:150+` |
| `napi_get_current_runtime` | N/A | 获取当前 Runtime | `jsnapi.cpp:200+` |

### 2. 模块加载与执行

| C++ API | JS API | 功能 | 绑定文件 |
|---------|--------|------|----------|
| `napi_load_module` | `loadModule()` | 加载 ABC 模块 | `jsnapi.cpp:500+` |
| `napi_load_module_with_info` | `loadModule(info)` | 带信息加载 | `jsnapi.cpp:550+` |
| `napi_run_generator` | N/A | 运行生成器 | `jsnapi.cpp:600+` |

### 3. 值创建与转换

| C++ API | JS 类型 | 功能 | 绑定文件 |
|---------|--------|------|----------|
| `napi_create_int32` | number | 创建整数 | `jsnapi.cpp:700+` |
| `napi_create_uint32` | number | 创建无符号整数 | `jsnapi.cpp:750+` |
| `napi_create_double` | number | 创建双精度浮点 | `jsnapi.cpp:800+` |
| `napi_create_string_utf8` | string | 创建字符串 | `jsnapi.cpp:850+` |
| `napi_create_object` | object | 创建对象 | `jsnapi.cpp:900+` |
| `napi_create_array` | array | 创建数组 | `jsnapi.cpp:950+` |
| `napi_create_arraybuffer` | ArrayBuffer | 创建数组缓冲区 | `jsnapi.cpp:1000+` |
| `napi_create_typedarray` | TypedArray | 创建类型化数组 | `jsnapi.cpp:1050+` |
| `napi_create_function` | function | 创建函数 | `jsnapi.cpp:1100+` |

### 4. 值属性操作

| C++ API | 功能 | 绑定文件 |
|---------|------|----------|
| `napi_get_named_property` | 获取命名属性 | `jsnapi.cpp:1200+` |
| `napi_set_named_property` | 设置命名属性 | `jsnapi.cpp:1250+` |
| `napi_get_property` | 获取属性 | `jsnapi.cpp:1300+` |
| `napi_set_property` | 设置属性 | `jsnapi.cpp:1350+` |
| `napi_delete_property` | 删除属性 | `jsnapi.cpp:1400+` |

### 5. 函数调用

| C++ API | 功能 | 绑定文件 |
|---------|------|----------|
| `napi_call_function` | 调用 JS 函数 | `jsnapi.cpp:1500+` |
| `napi_create_function` | 创建 JS 函数 | `jsnapi.cpp:1550+` |
| `napi_get_cb_info` | 获取回调参数 | `jsnapi.cpp:1600+` |

### 6. 异步操作

| C++ API | JS API | 功能 | 绑定文件 |
|---------|--------|------|----------|
| `napi_create_async_work` | N/A | 创建异步工作 | `jsnapi.cpp:1700+` |
| `napi_delete_async_work` | N/A | 删除异步工作 | `jsnapi.cpp:1750+` |
| `napi_queue_async_work` | N/A | 队列异步工作 | `jsnapi.cpp:1800+` |
| `napi_cancel_async_work` | N/A | 取消异步工作 | `jsnapi.cpp:1850+` |

### 7. 线程安全回调

| C++ API | 功能 | 绑定文件 |
|---------|------|----------|
| `napi_create_threadsafe_function` | 创建线程安全函数 | `jsnapi.cpp:1900+` |
| `napi_call_threadsafe_function` | 调用线程安全函数 | `jsnapi.cpp:1950+` |
| `napi_release_threadsafe_function` | 释放线程安全函数 | `jsnapi.cpp:2000+` |

### 8. 错误处理

| C++ API | 功能 | 绑定文件 |
|---------|------|----------|
| `napi_throw` | 抛出异常 | `jsnapi.cpp:2100+` |
| `napi_throw_error` | 抛出错误 | `jsnapi.cpp:2150+` |
| `napi_throw_type_error` | 抛出类型错误 | `jsnapi.cpp:2200+` |
| `napi_throw_range_error` | 抛出范围错误 | `jsnapi.cpp:2250+` |
| `napi_is_error` | 检查是否为错误 | `jsnapi.cpp:2300+` |

## 参数校验机制

### 类型校验

N-API 对所有输入参数进行严格的类型校验，校验失败时返回错误码。

```cpp
// 示例：检查是否为 Object 类型
napi_status napi_is_object(napi_env env, napi_value value, bool *result) {
    // 内部实现会检查 value 的类型标记
    // 如果不是对象类型，返回 napi_object_expected
}
```

### 边界校验

- **字符串长度**：校验 UTF-8 编码长度不超过 `NAPI_MAX_STRING_LENGTH`
- **数组索引**：校验索引范围在 `[0, INT32_MAX]` 内
- **对象键**：校验键名不为空且长度合理

### 错误码定义

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `napi_ok` | 0 | 成功 |
| `napi_invalid_arg` | 1 | 无效参数 |
| `napi_object_expected` | 2 | 期望对象类型 |
| `napi_string_expected` | 3 | 期望字符串类型 |
| `napi_name_expected` | 4 | 期望属性名 |
| `napi_function_expected` | 5 | 期望函数类型 |
| `napi_number_expected` | 6 | 期望数值类型 |
| `napi_boolean_expected` | 7 | 期望布尔类型 |
| `napi_array_expected` | 8 | 期望数组类型 |
| `napi_generic_failure` | 9 | 通用失败 |
| `napi_pending_exception` | 10 | 挂起异常 |
| `napi_cancelled` | 11 | 已取消 |
| `napi_escape_called_twice` | 12 | 重复调用 escape |
| `napi_handle_scope_mismatch` | 13 | 句柄作用域不匹配 |

## JS API 导出清单

### @ohos 命名空间

以下 API 通过 N-API 导出到 `@ohos` 命名空间：

| 模块 | JS API | C++ 实现 |
|------|--------|----------|
| `@ohos.buffer` | Buffer | `js_api_buffer.cpp` |
| `@ohos.arrayList` | ArrayList | `js_api_arraylist.cpp` |
| `@ohos.linkedList` | LinkedList | `js_api_linked_list.cpp` |
| `@ohos.hashMap` | HashMap | `js_api_hashmap.cpp` |
| `@ohos.hashSet` | HashSet | `js_api_hashset.cpp` |
| `@ohos.treeMap` | TreeMap | `js_api_tree_map.cpp` |
| `@ohos.treeSet` | TreeSet | `js_api_tree_set.cpp` |
| `@ohos.vector` | Vector | `js_api_vector.cpp` |
| `@ohos.deque` | Deque | `js_api_deque.cpp` |
| `@ohos.stack` | Stack | `js_api_stack.cpp` |
| `@ohos.queue` | Queue | `js_api_queue.cpp` |
| `@ohos.list` | List | `js_api_list.cpp` |
| `@ohos.plainArray` | PlainArray | `js_api_plain_array.cpp` |
| `@ohos.lightWeightMap` | LightWeightMap | `js_api_lightweightmap.cpp` |
| `@ohos.lightWeightSet` | LightWeightSet | `js_api_lightweightset.cpp` |
| `@ohos.bitVector` | BitVector | `js_api_bitvector.cpp` |

### containers 命名空间

以下 API 通过 N-API 导出到 `containers` 命名空间：

| 模块 | JS API | C++ 实现 |
|------|--------|----------|
| `containers.ArrayList` | ArrayList | `containers_arraylist.cpp` |
| `containers.Vector` | Vector | `containers_vector.cpp` |
| `containers.Deque` | Deque | `containers_deque.cpp` |
| `containers.Queue` | Queue | `containers_queue.cpp` |
| `containers.Stack` | Stack | `containers_stack.cpp` |
| `containers.List` | List | `containers_list.cpp` |
| `containers.LinkedList` | LinkedList | `containers_linked_list.cpp` |
| `containers.HashMap` | HashMap | `containers_hashmap.cpp` |
| `containers.HashSet` | HashSet | `containers_hashset.cpp` |
| `containers.TreeMap` | TreeMap | `containers_treemap.cpp` |
| `containers.TreeSet` | TreeSet | `containers_treeset.cpp` |
| `containers.LightWeightMap` | LightWeightMap | `containers_lightweightmap.cpp` |
| `containers.LightWeightSet` | LightWeightSet | `containers_lightweightset.cpp` |
| `containers.PlainArray` | PlainArray | `containers_plainarray.cpp` |
| `containers.BitVector` | BitVector | `containers_bitvector.cpp` |

## 相关文档

- [项目概览](01_Project_Overview.md)
- [目录结构](02_Directory_Structure.md)
- [架构说明](03_Architecture.md)
- [内部 API](05_Inner_API.md)
