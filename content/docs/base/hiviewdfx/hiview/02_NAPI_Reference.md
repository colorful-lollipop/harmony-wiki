# N-API 接口参考

## 概述

Hiview 对外提供以下 N-API 模块供 JavaScript/ArkTS 调用：

| 模块名 | JS 命名空间 | C++ 实现路径 |
|--------|------------|--------------|
| `faultLogger` | `@ohos.faultLogger` | `plugins/faultlogger/interfaces/js/napi/` |
| `logLibrary` | `@ohos.logLibrary` | `interfaces/js/napi/` |

> 证据: `bundle.json:122-126` sub_component 定义

---

## faultLogger 模块

### 模块信息

| 属性 | 值 |
|------|-----|
| 模块名 | `faultLogger` |
| N-API 版本 | 1 |
| 实现文件 | `plugins/faultlogger/interfaces/js/napi/napi_faultlogger.cpp` |
| 注册方式 | `napi_module_register(&_stateRegistryModule)` |

### 导出 API 清单

#### 1. querySelfFaultLog

```typescript
function querySelfFaultLog(type: FaultType, callback?: AsyncCallback<FaultLogInfo[]>): void
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `type` | `FaultType` | 故障类型枚举 |
| `callback` | `AsyncCallback<FaultLogInfo[]>` | 异步回调，返回故障日志列表 |

**返回值**: `void`

**错误码**:

| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| 401 | 参数错误 |
| 16500050 | 内部错误 |

**调用位置**: `napi_faultlogger.cpp:QuerySelfFaultLog()`

#### 2. addFaultLog

```typescript
function addFaultLog(now: number, logType: FaultType, module: string, summary: string): number | 类型 | 说明
```

| 参数 |
|------|------|------|
| `now` | `number` | 时间戳（毫秒） |
| `logType` | `FaultType` | 故障类型 |
| `module` | `string` | 模块名 |
| `summary` | `string` | 摘要信息 |

**返回值**: `number` - 0 表示成功，非 0 表示失败

**调用位置**: `napi_faultlogger.cpp:AddFaultLog()`

#### 3. query (新版)

```typescript
function query(type: FaultType, callback?: AsyncCallback<FaultLogInfo[]>): void
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `type` | `FaultType` | 故障类型 |
| `callback` | `AsyncCallback<FaultLogInfo[]>` | 异步回调 |

**调用位置**: `napi_faultlogger.cpp:Query()`

### 枚举定义

#### FaultType

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| `NO_SPECIFIC` | 0 | 未指定类型 |
| `CPP_CRASH` | 1 | C++ 崩溃 |
| `JS_CRASH` | 2 | JavaScript 崩溃 |
| `APP_FREEZE` | 3 | 应用冻结 |

### 类定义

#### FaultLogInfo

| 属性 | 类型 | 说明 |
|------|------|------|
| `pid` | `number` | 进程 ID |
| `uid` | `number` | 用户 ID |
| `type` | `FaultType` | 故障类型 |
| `timestamp` | `number` | 时间戳 |
| `reason` | `string` | 故障原因 |
| `module` | `string` | 模块名 |
| `summary` | `string` | 摘要 |
| `fullLog` | `string` | 完整日志 |

### 调用链

```
JS: faultLogger.querySelfFaultLog(type, callback)
    ↓
napi_faultlogger.cpp: QuerySelfFaultLog()
    ↓
hiview_remote_service.cpp: GetHiViewRemoteService()
    ↓ (IPC)
FaultloggerServiceStub::QuerySelfFaultLog()
    ↓
FaultLoggerImpl::QuerySelfFaultLog()
```

---

## logLibrary 模块

### 模块信息

| 属性 | 值 |
|------|-----|
| 模块名 | `logLibrary` |
| N-API 版本 | 1 |
| 实现文件 | `interfaces/js/napi/src/napi_hiview_js.cpp` |
| 注册方式 | `napi_module_register(&_module)` |
| 权限要求 | 系统应用权限 |

### 导出 API 清单

#### 1. list

```typescript
function list(logType: string): string[]
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `logType` | `string` | 日志类型 |

**返回值**: `string[]` - 日志文件列表

**调用位置**: `napi_hiview_js.cpp: List()`

#### 2. copy

```typescript
function copy(logType: string, logName: string, destDir: string, callback?: AsyncCallback<number>): void
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `logType` | `string` | 日志类型 |
| `logName` | `string` | 日志文件名 |
| `destDir` | `string` | 目标目录 |
| `callback` | `AsyncCallback<number>` | 异步回调 |

**返回值**: `void`

**调用位置**: `napi_hiview_js.cpp: Copy()`

#### 3. move

```typescript
function move(logType: string, logName: string, destDir: string, callback?: AsyncCallback<number>): void
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `logType` | `string` | 日志类型 |
| `logName` | `string` | 日志文件名 |
| `destDir` | `string` | 目标目录 |
| `callback` | `AsyncCallback<number>` | 异步回调 |

**返回值**: `void`

**调用位置**: `napi_hiview_js.cpp: Move()`

#### 4. remove

```typescript
function remove(logType: string, logName: string): boolean
```

| 参数 | 类型 | 说明 |
|------|------|------|
| `logType` | `string` | 日志类型 |
| `logName` | `string` | 日志文件名 |

**返回值**: `boolean` - true 表示成功

**调用位置**: `napi_hiview_js.cpp: Remove()`

### 权限检查

logLibrary API 需要以下权限之一：

| 权限 | 说明 |
|------|------|
| `ohos.permission.READ_HIVIEW_SYSTEM` | 读取 HIVIEW 系统 |
| `ohos.permission.WRITE_HIVIEW_SYSTEM` | 写入 HIVIEW 系统 |

**调用位置**: `hiview_napi_util.cpp: IsSystemAppCall()`

### 调用链

```
JS: logLibrary.list(logType)
    ↓
napi_hiview_js.cpp: List()
    ↓
hiview_service_agent.cpp: List()
    ↓
IPC Proxy → HiviewServiceAbility
    ↓
HiviewServiceAbility::ListFiles()
```

---

## Extension 接口

### FaultLogExtensionContext

**模块名**: `hiviewdfx.FaultLogExtensionContext`

**实现文件**:
- C++: `plugins/faultlogger/interfaces/js/napi/fault_log_extension_context/fault_log_extension_context_module.cpp`
- JS: `plugins/faultlogger/interfaces/js/napi/fault_log_extension_context/fault_log_extension_context.js`

**注册方式**: `NativeModuleManager::Register()`

**继承**: `ExtensionContext`

### FaultLogExtensionAbility

**模块名**: `hiviewdfx.FaultLogExtensionAbility`

**实现文件**:
- C++: `plugins/faultlogger/interfaces/js/napi/fault_log_extension/fault_log_extension_ability_module.cpp`
- JS: `plugins/faultlogger/interfaces/js/napi/fault_log_extension/fault_log_extension_ability.js`

**注册方式**: `NativeModuleManager::Register()`

**继承**: `ExtensionAbility`

**生命周期方法**:
- `onFaultReportReady()` - 故障报告就绪
- `onConnect()` - 连接建立
- `onDisconnect()` - 断开连接

---

## N-API 工具类

### napi_faultlogger 工具函数

**文件**: `plugins/faultlogger/interfaces/js/napi/napi_util.cpp`

| 函数 | 功能 |
|------|------|
| `CreateErrorMessage()` | 创建错误消息 |
| `CreateUndefined()` | 创建 undefined 值 |
| `SetPropertyInt32/Int64/StringUtf8()` | 设置对象属性 |
| `IsMatchType()` | 类型检查 |
| `CreateString()` | 创建字符串 |
| `ThrowError()` | 抛出错误 |

### hiview_napi_util 工具函数

**文件**: `interfaces/js/napi/src/hiview_napi_util.cpp`

| 函数 | 功能 |
|------|------|
| `CreateUndefined/Int32Value/Int64Value/StringValue()` | 创建 N-API 值 |
| `CreateErrorByRet()` | 根据返回码创建错误 |
| `IsMatchType()` | 类型匹配检查 |
| `ParseStringValue()` | 解析字符串参数 |
| `GenerateFileInfoResult()` | 生成文件信息结果数组 |
| `ThrowParamTypeError()` | 参数类型错误 |
| `ThrowParamContentError()` | 参数内容错误 |
| `ThrowErrorByCode()` | 根据错误码抛出错误 |
| `IsSystemAppCall()` | 检查系统应用调用 |

---

## 错误码定义

### FaultLogger 错误码

**文件**: `plugins/faultlogger/interfaces/js/napi/napi_error.h`

| 错误码 | 说明 |
|--------|------|
| 16500000 | 内部错误 |
| 16500050 | 内存分配失败 |
| 16500100 | 文件不存在 |
| 16500101 | 文件读取失败 |

### Hiview 服务错误码

**文件**: `adapter/service/common/include/hiview_err_code.h`

| 错误码 | 说明 |
|--------|------|
| 0 | 成功 |
| -1 | 通用错误 |
| 1 | 参数错误 |
| 2 | 权限错误 |
| 3 | 文件不存在 |
| 4 | 文件操作失败 |
