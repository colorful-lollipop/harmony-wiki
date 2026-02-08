# N-API 绑定

## 概述

`ets2panda/bindings/native/` 模块提供 Native ↔ JavaScript 互操作能力，基于 Node.js N-API 实现。

## 模块结构

```
bindings/native/
├── src/
│   ├── common-interop.cpp        # Promise/Async 处理
│   ├── convertors-napi.cpp       # 类型转换 (JS ↔ C++)
│   └── win-dynamic-node.cpp      # 动态模块加载
└── include/
    └── *.h                       # 头文件
```

## N-API 注册机制

### 模块注册 (convertors-napi.cpp:25-32)

```cpp
// 使用 NAPI_MODULE 宏注册模块
NAPI_MODULE(modname, InitModule)
```

**证据**: `ets2panda/bindings/native/src/convertors-napi.cpp:25-32`

### 导出函数映射

| N-API 函数 | 用途 |
|------------|------|
| `napi_create_promise` | 创建 Promise |
| `napi_resolve_deferred` | 解决 Promise |
| `napi_reject_deferred` | 拒绝 Promise |
| `napi_create_threadsafe_function` | 创建线程安全函数 |
| `napi_call_threadsafe_function` | 调用线程安全函数 |
| `napi_create_function` | 创建 JS 函数 |
| `napi_define_properties` | 定义对象属性 |
| `napi_set_named_property` | 设置命名属性 |
| `napi_get_property` | 获取属性 |

**证据**: `ets2panda/bindings/native/src/win-dynamic-node.cpp:25-66`

## 类型转换

### JS → C++ 转换

| 函数 | 描述 | 错误处理 |
|------|------|----------|
| `GetBoolean(env, value)` | 获取 Boolean | 类型检查 |
| `GetInt32(env, value)` | 获取 Int32 | 类型检查 + throw |
| `GetUInt32(env, value)` | 获取 UInt32 | 类型检查 |
| `GetFloat32(env, value)` | 获取 Float32 | 类型检查 |
| `GetFloat64(env, value)` | 获取 Float64 | 类型检查 |
| `GetString(env, value)` | 获取 String | 类型检查 + 长度验证 |
| `GetPointer(env, value)` | 获取指针 (BigInt) | 类型检查 + 范围检查 |
| `GetInt64(env, value)` | 获取 Int64 (BigInt) | 类型检查 + 范围检查 |

**证据**: `ets2panda/bindings/native/src/convertors-napi.cpp:36-168`

### C++ → JS 转换

| 函数 | 描述 |
|------|------|
| `MakeBoolean(env, value)` | 创建 Boolean |
| `MakeInt32(env, value)` | 创建 Int32 |
| `MakeUInt32(env, value)` | 创建 UInt32 |
| `MakeFloat32(env, value)` | 创建 Float32 |
| `MakeFloat64(env, value)` | 创建 Double |
| `MakeString(env, value)` | 创建 String |
| `MakePointer(env, value)` | 创建指针 (BigInt) |
| `MakeObject(env, value)` | 创建 Object |
| `MakeVoid(env)` | 创建 undefined |

**证据**: `ets2panda/bindings/native/src/convertors-napi.cpp:174-247`

## Promise 支持

### Promise 创建流程

```
napi_create_promise()
        │
        ▼
┌───────────────────┐
│ Create deferred   │
│ context           │
└─────────┬─────────┘
          │
          ▼
napi_create_threadsafe_function()
        │
        ▼
Return promise to JS
```

**证据**: `ets2panda/bindings/native/src/common-interop.cpp:262-312`

### 异步处理

- 使用 `napi_threadsafe_function` 实现线程安全异步调用
- 支持 `napi_tsfn_nonblocking` 和 `napi_tsfn_release` 模式

## 错误处理

### 异常抛出

```cpp
// 抛出类型错误
napi_throw_error(env, nullptr, "Expected Number");

// 抛出 TypeError
napi_create_type_error(env, code, msg, &result);
```

**证据**: `ets2panda/bindings/native/src/convertors-napi.cpp:65-76`

### 错误码处理

所有 N-API 调用检查返回值：

```cpp
napi_status status = napi_get_value_int32(env, value, &result);
TS_NAPI_THROW_IF_FAILED(env, status, napi_undefined);
```

## 使用示例

### 创建导出函数

```cpp
static napi_value MyFunction(napi_env env, napi_callback_info info) {
    // 获取参数
    size_t argc = 1;
    napi_value argv[1];
    napi_get_cb_info(env, info, &argc, argv, nullptr, nullptr);

    // 类型转换
    KInt value = GetInt32(env, argv[0]);

    // 处理逻辑
    // ...

    // 返回结果
    return MakeInt32(env, result);
}
```

## 相关文档

- [架构详解](07_Architecture.md)
- [编译产物与运行时](06_Build_Outputs.md)
