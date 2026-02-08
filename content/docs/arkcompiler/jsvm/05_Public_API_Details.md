# 对外 API 详细文档

> API 清单表、参数、返回值、错误码、权限

## 目的与适用范围

### 目的
本文档提供 JSVM 对外 API 的详细信息，包括 API 清单、参数、返回值、错误码、权限等。

### 适用范围
- 需要使用 JSVM-API 的开发者
- 需要进行 API 审查的开发者
- 需要进行安全评审的开发者

---

## API 清单表

### VM 生命周期管理（VM Lifecycle）

| API 名称 | 版本 | 同步/异步 | 实现文件 | 行号 |
|---------|------|-----------|---------|------|
| `OH_JSVM_Init` | 11 | 同步 | `src/js_native_api_v8.cpp` | 1059 |
| `OH_JSVM_CreateVM` | 11 | 同步 | `src/js_native_api_v8.cpp` | 1094 |
| `OH_JSVM_DestroyVM` | 11 | 同步 | `src/js_native_api_v8.cpp` | 1133 |
| `OH_JSVM_OpenVMScope` | 11 | 同步 | `src/js_native_api_v8.cpp` | 1159 |
| `OH_JSVM_CloseVMScope` | 11 | 同步 | `src/js_native_api_v8.cpp` | 1167 |
| `OH_JSVM_CreateEnv` | 11 | 同步 | `src/js_native_api_v8.cpp` | 1174 |
| `OH_JSVM_CreateEnvFromSnapshot` | 11 | 同步 | `src/js_native_api_v8.cpp` | 1233 |
| `OH_JSVM_DestroyEnv` | 11 | 同步 | `src/js_native_api_v8.cpp` | 1253 |
| `OH_JSVM_OpenEnvScope` | 11 | 同步 | `src/js_native_api_v8.cpp` | 1260 |
| `OH_JSVM_CloseEnvScope` | 11 | 同步 | `src/js_native_api_v8.cpp` | 1267 |
| `OH_JSVM_GetVM` | 12 | 同步 | `src/js_native_api_v8.cpp` | 1088 |

**证据位置**:
- `interface/kits/jsvm.h:323-486` - API 声明
- `src/js_native_api_v8.cpp:1059-1267` - API 实现

### 代码编译与执行（Code Compilation & Execution）

| API 名称 | 版本 | 同步/异步 | 实现文件 | 行号 |
|---------|------|-----------|---------|------|
| `OH_JSVM_CompileScript` | 11 | 同步 | `src/js_native_api_v8.cpp` | 1274 |
| `OH_JSVM_CompileScriptWithOrigin` | 12 | 同步 | `src/js_native_api_v8.cpp` | 1361 |
| `OH_JSVM_CompileScriptWithOptions` | 12 | 同步 | `src/js_native_api_v8.cpp` | 1498 |
| `OH_JSVM_RunScript` | 11 | 同步 | `src/js_native_api_v8.cpp` | 1550 |
| `OH_JSVM_CreateCodeCache` | 11 | 同步 | `src/js_native_api_v8.cpp` | 1528 |
| `OH_JSVM_CreateSnapshot` | 11 | 同步 | `src/js_native_api_v8.cpp` | 1608 |
| `OH_JSVM_JsonParse` | 12 | 同步 | `src/js_native_api_v8.cpp` | 1575 |
| `OH_JSVM_JsonStringify` | 12 | 同步 | `src/js_native_api_v8.cpp` | 1592 |

**证据位置**:
- `interface/kits/jsvm.h:489-1608` - API 声明
- `src/js_native_api_v8.cpp:1274-1608` - API 实现

### 值操作（Value Operations）

#### 值创建（Value Creation）

| API 名称 | 版本 | 同步/异步 | 实现文件 | 行号 |
|---------|------|-----------|---------|------|
| `OH_JSVM_CreateUndefined` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_CreateNull` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_CreateBoolean` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_CreateInt32` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_CreateUint32` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_CreateInt64` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_CreateDouble` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_CreateBigInt` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_CreateString` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_CreateArray` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_CreateArrayWithLength` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_CreateArraybuffer` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_CreateTypedarray` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_CreateDataview` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_CreateObject` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_CreateSymbol` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_CreateExternal` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_CreateDate` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_CreateProxy` | 18 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |

**证据位置**:
- `interface/kits/jsvm.h:907-1074` - API 声明
- `src/js_native_api_v8.cpp` - API 实现（待定位）

#### 值检查（Value Checking）

| API 名称 | 版本 | 同步/异步 | 实现文件 | 行号 |
|---------|------|-----------|---------|------|
| `OH_JSVM_Typeof` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_IsUndefined` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_IsNull` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_IsBoolean` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_IsNumber` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_IsString` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_IsSymbol` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_IsObject` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_IsFunction` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_IsExternal` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_IsBigInt` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_IsArray` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_IsArrayBuffer` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_IsTypedArray` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_IsDataview` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_IsError` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_IsPromise` | 11 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |
| `OH_JSVM_IsProxy` | 18 | 同步 | `src/js_native_api_v8.cpp` | [待定位] |

**证据位置**:
- `interface/kits/jsvm.h:1176-1285` - API 声明
- `src/js_native_api_v8.cpp` - API 实现（待定位）

### 调试与性能分析（Debugging & Profiling）

| API 名称 | 版本 | 同步/异步 | 实现文件 | 行号 |
|---------|------|-----------|---------|------|
| `OH_JSVM_GetVMInfo` | 11 | 同步 | `src/js_native_api_v8.cpp` | 1641 |
| `OH_JSVM_MemoryPressureNotification` | 11 | 同步 | `src/js_native_api_v8.cpp` | 1650 |
| `OH_JSVM_GetHeapStatistics` | 12 | 同步 | `src/js_native_api_v8.cpp` | 1661 |
| `OH_JSVM_StartCpuProfiler` | 12 | 同步 | `src/js_native_api_v8.cpp` | 1682 |
| `OH_JSVM_StopCpuProfiler` | 12 | 同步 | `src/js_native_api_v8.cpp` | 1693 |
| `OH_JSVM_TakeHeapSnapshot` | 12 | 同步 | `src/js_native_api_v8.cpp` | 1708 |
| `OH_JSVM_OpenInspector` | 12 | 同步 | `src/inspector/js_native_api_v8_inspector.cpp` | [待定位] |
| `OH_JSVM_CloseInspector` | 12 | 同步 | `src/inspector/js_native_api_v8_inspector.cpp` | [待定位] |
| `OH_JSVM_WaitForDebugger` | 12 | 同步 | `src/inspector/js_native_api_v8_inspector.cpp` | [待定位] |

**证据位置**:
- `interface/kits/jsvm.h:1641-1774` - API 声明
- `src/js_native_api_v8.cpp` / `src/inspector/js_native_api_v8_inspector.cpp` - API 实现

---

## 关键 API 详细说明

### 1. VM 生命周期管理

#### OH_JSVM_Init

**功能**: 初始化 JSVM 全局状态。

**签名**:
```c
JSVM_Status OH_JSVM_Init(const JSVM_InitOptions* options);
```

**参数**:
- `options`: 初始化选项，可以为 NULL

**返回值**:
- `JSVM_OK`: 成功
- `JSVM_GENERIC_FAILURE`: 失败（已经初始化）

**权限**: 无特殊权限

**错误码**:
- `JSVM_GENERIC_FAILURE`: 当前进程已经完成了 JSVM 初始化，无需重复执行

**证据位置**:
- `interface/kits/jsvm.h:323` - API 声明
- `src/js_native_api_v8.cpp:1059` - API 实现

#### OH_JSVM_CreateVM

**功能**: 创建一个新的 VM 实例。

**签名**:
```c
JSVM_Status OH_JSVM_CreateVM(const JSVM_CreateVMOptions* options, JSVM_VM* result);
```

**参数**:
- `options`: 创建选项，包含内存限制、快照数据等
- `result`: 输出参数，返回新创建的 VM 实例

**返回值**:
- `JSVM_OK`: 成功
- 其他错误码

**权限**: 无特殊权限

**错误码**:
- `JSVM_INVALID_ARG`: 参数无效
- `JSVM_GENERIC_FAILURE`: 创建失败

**证据位置**:
- `interface/kits/jsvm.h:333` - API 声明
- `src/js_native_api_v8.cpp:1094` - API 实现

### 2. 代码编译与执行

#### OH_JSVM_CompileScript

**功能**: 编译 JavaScript 代码。

**签名**:
```c
JSVM_Status OH_JSVM_CompileScript(JSVM_Env env,
                               JSVM_Value script,
                               const uint8_t* cachedData,
                               size_t cacheDataLength,
                               bool eagerCompile,
                               bool* cacheRejected,
                               JSVM_Script* result);
```

**参数**:
- `env`: 环境
- `script`: JavaScript 代码字符串
- `cachedData`: 可选的代码缓存数据
- `cacheDataLength`: 缓存数据长度
- `eagerCompile`: 是否急切编译
- `cacheRejected`: 输出参数，缓存是否被拒绝
- `result`: 输出参数，返回编译后的脚本

**返回值**:
- `JSVM_OK`: 成功
- 其他错误码

**权限**: 无特殊权限

**错误码**:
- `JSVM_INVALID_ARG`: 参数无效
- `JSVM_STRING_EXPECTED`: script 不是字符串
- `JSVM_PENDING_EXCEPTION`: 编译时出现异常

**证据位置**:
- `interface/kits/jsvm.h:501-507` - API 声明
- `src/js_native_api_v8.cpp:1274` - API 实现

### 3. 调试与性能分析

#### OH_JSVM_OpenInspector

**功能**: 打开 Inspector 调试器。

**签名**:
```c
JSVM_Status OH_JSVM_OpenInspector(JSVM_Env env, const char* host, uint16_t port);
```

**参数**:
- `env`: 环境
- `host`: 主机地址（如 "0.0.0.0"）
- `port`: 端口号（如 9229）

**返回值**:
- `JSVM_OK`: 成功
- 其他错误码

**权限**: 需要网络权限（TODO: 需确认具体权限）

**错误码**:
- `JSVM_INVALID_ARG`: 参数无效
- `JSVM_GENERIC_FAILURE`: 打开失败

**证据位置**:
- `interface/kits/jsvm.h:1719` - API 声明
- `src/inspector/js_native_api_v8_inspector.cpp` - API 实现（待定位）

---

## 参数校验与错误码

### 参数校验策略

JSVM API 对参数进行严格的类型检查：

1. **NULL 指针检查**: 所有指针参数必须非 NULL（除非明确允许）
2. **类型检查**: 值类型必须符合 API 要求
3. **范围检查**: 数组索引、缓冲区偏移等必须在有效范围内

**证据位置**:
- `src/js_native_api_v8.cpp` - 各 API 的参数校验逻辑

### 常见错误码

| 错误码 | 含义 | 常见原因 |
|--------|------|---------|
| `JSVM_OK` | 成功 | 无 |
| `JSVM_INVALID_ARG` | 参数无效 | NULL 指针、无效参数值 |
| `JSVM_OBJECT_EXPECTED` | 期望对象 | 传递了非对象类型 |
| `JSVM_STRING_EXPECTED` | 期望字符串 | 传递了非字符串类型 |
| `JSVM_FUNCTION_EXPECTED` | 期望函数 | 传递了非函数类型 |
| `JSVM_NUMBER_EXPECTED` | 期望数字 | 传递了非数字类型 |
| `JSVM_BOOLEAN_EXPECTED` | 期望布尔 | 传递了非布尔类型 |
| `JSVM_ARRAY_EXPECTED` | 期望数组 | 传递了非数组类型 |
| `JSVM_GENERIC_FAILURE` | 通用失败 | 未知错误 |
| `JSVM_PENDING_EXCEPTION` | 有待处理的异常 | JavaScript 异常未处理 |

**证据位置**:
- `interface/kits/jsvm_types.h:277-313` - 错误码定义

---

## 同步/异步模式

### 同步 API

大部分 JSVM API 都是同步的，调用后立即返回结果。

**示例**:
- `OH_JSVM_CreateEnv()` - 同步创建
- `OH_JSVM_CompileScript()` - 同步编译
- `OH_JSVM_RunScript()` - 同步执行

**证据位置**: `interface/kits/jsvm.h` - 所有 API 声明

### 异步支持

JSVM 提供了异步操作的机制，但不通过 API 签名直接体现：

#### Promise

Promise 是 JavaScript 的异步机制，JSVM API 支持创建和操作 Promise。

**相关 API**:
- `OH_JSVM_CreatePromise()` - 创建 Promise
- `OH_JSVM_ResolvePromise()` - 解决 Promise
- `OH_JSVM_RejectPromise()` - 拒绝 Promise

**证据位置**: `interface/kits/jsvm.h` - Promise 相关 API

#### Microtasks

Microtasks 是 JavaScript 的微任务队列，JSVM 支持控制 Microtask 的调度。

**相关 API**:
- `OH_JSVM_SetMicrotaskPolicy()` - 设置 Microtask 策略
- `OH_JSVM_PerformMicrotaskCheckpoint()` - 执行 Microtask checkpoint

**证据位置**:
- `interface/kits/jsvm.h:344` - `OH_JSVM_SetMicrotaskPolicy()`
- `interface/kits/jsvm.h:1770` - `OH_JSVM_PerformMicrotaskCheckpoint()`

---

## 权限与前置条件

### 权限要求

JSVM API 本身不需要特殊权限，但某些功能可能需要：

| 功能 | 权限要求 | 证据 |
|------|----------|------|
| 文件 I/O | 依赖平台层 | `src/platform/platform_ohos.cpp` |
| 网络 I/O | 需要网络权限 | `bundle.json:34` - nghttp2 |
| Inspector | 需要网络权限 | `src/inspector/inspector_socket_server.cpp` |

**证据位置**:
- `bundle.json:24-38` - 依赖组件
- `src/platform/` - 平台层实现

### 前置条件

使用 JSVM API 前需要满足以下条件：

1. **已初始化 JSVM**: 调用 `OH_JSVM_Init()`
2. **已创建 VM**: 调用 `OH_JSVM_CreateVM()`
3. **已创建 Environment**: 调用 `OH_JSVM_CreateEnv()`

**证据位置**:
- `interface/kits/jsvm.h:323-436` - 初始化流程

---

## 相关链接

- [对外 API 总览](./03_Public_API_Overview.md) - API 分类
- [架构说明](./04_Architecture.md) - API 调用链
- [附录：调用链图](./appendix/Callgraphs.md) - 详细调用链
