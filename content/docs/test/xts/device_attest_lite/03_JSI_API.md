# JavaScript 接口 (JSI)

> **重要说明**: 本模块使用 **OpenHarmony ACELite JSI (JavaScript Interface)** 而非标准 N-API。
> JSI 是专为轻量级系统设计的 JS 绑定方案。

## 3.1 API 清单

| JS 函数名 | 类型 | Native 处理函数 | 行号 |
|-----------|------|-----------------|------|
| `getAttestStatus` | 异步 (Callback) | `NativeDeviceAttest::GetAttestResultInfoAsync` | `native_device_attest.cpp:210-213` |
| `getAttestStatusSync` | 同步 | `NativeDeviceAttest::GetAttestResultInfoSync` | `native_device_attest.cpp:181-208` |

**证据**: `interfaces/kit/js/src/native_device_attest.cpp:210-213, 181-208`

---

## 3.2 模块注册

### 注册点

```cpp
// 文件: interfaces/kit/js/src/native_device_attest.cpp:217-221

void InitDeviceAttestModule(JSIValue exports)
{
    JSI::SetModuleAPI(exports, "getAttestStatus", NativeDeviceAttest::GetAttestResultInfoAsync);
    JSI::SetModuleAPI(exports, "getAttestStatusSync", NativeDeviceAttest::GetAttestResultInfoSync);
}
```

**证据**: `native_device_attest.cpp:217-221`

### 调用链

```
JS 调用 getAttestStatus()
    ↓
JSI Runtime
    ↓
NativeDeviceAttest::GetAttestResultInfoAsync()
    ↓
ExecuteAsyncWork()
    ↓
JsAsyncWork::DispatchAsyncWork()
    ↓
ExecuteGetAttestResult()
    ↓
GetAttestStatus() [Inner API]
```

---

## 3.3 getAttestStatus (异步)

### JS 调用示例

```javascript
deviceAttest.getAttestStatus((err, result) => {
    if (err) {
        console.error(`Error: ${err.code}, ${err.message}`);
        return;
    }
    console.log(`AuthResult: ${result.authResult}`);
    console.log(`SoftwareResult: ${result.softwareResult}`);
    console.log(`Ticket: ${result.ticket}`);
});
```

### Native 实现

```cpp
// interfaces/kit/js/src/native_device_attest.cpp:210-213

JSIValue NativeDeviceAttest::GetAttestResultInfoAsync(const JSIValue thisVal, const JSIValue *args, uint8_t argsNum)
{
    HILOGI("[GetAttestResultInfoAsync] In.");
    return ExecuteAsyncWork(thisVal, args, argsNum, ExecuteGetAttestResult);
}
```

### 参数说明

| 参数 | 类型 | 说明 |
|------|------|------|
| thisVal | JSIValue | JS this 上下文 |
| args | JSIValue* | JS 参数数组 |
| argsNum | uint8_t | 参数数量 |

### 异步工作执行

```cpp
// interfaces/kit/js/src/native_device_attest.cpp:160-179

void ExecuteGetAttestResult(void* data)
{
    FuncParams* params = reinterpret_cast<FuncParams *>(data);
    // ...
    JSIValue result = JSI::CreateObject();
    int32_t ret = GetAttestResultInfo(&result);
    if (ret != DEVATTEST_SUCCESS) {
        FailCallBack(thisVal, args, ret);
    } else {
        SuccessCallBack(thisVal, args, result);
    }
    // ...
}
```

---

## 3.4 getAttestStatusSync (同步)

### JS 调用示例

```javascript
const result = deviceAttest.getAttestStatusSync();
if (result) {
    console.log(`AuthResult: ${result.authResult}`);
    console.log(`SoftwareResult: ${result.softwareResult}`);
}
```

### Native 实现

```cpp
// interfaces/kit/js/src/native_device_attest.cpp:181-208

JSIValue NativeDeviceAttest::GetAttestResultInfoSync(const JSIValue thisVal, const JSIValue *args, uint8_t argsNum)
{
    HILOGI("[GetAttestResultInfoSync] In.");
    AttestResultInfo attestResultInfo = { 0 };
    attestResultInfo.ticket = NULL;
    
    int32_t ret = GetAttestStatus(&attestResultInfo);
    if (ret != DEVATTEST_SUCCESS) {
        return GetJsiErrorMessage(ret);
    }
    
    // 构建 JS 对象返回
    JSIValue result = JSI::CreateObject();
    ret = SetJsResult(&result, &attestResultInfo);
    // ...
    return result;
}
```

**证据**: `native_device_attest.cpp:181-208`

### 执行耗时

> **注意**: 同步接口耗时约 10ms
> 
> 证据: `interfaces/innerkits/devattest_interface.h:34`

---

## 3.5 返回值结构

### AttestResult 对象

| 属性 | 类型 | 描述 |
|------|------|------|
| authResult | number | 认证结果状态码 |
| softwareResult | number | 软件验证结果 |
| softwareResultDetail | number[] | 详细结果数组 (5个元素) |
| ticket | string | 认证票据字符串 |

### 类型定义 (TypeScript 示意)

```typescript
interface AttestResult {
    authResult: number;
    softwareResult: number;
    softwareResultDetail: number[];
    ticket: string;
}

interface AttestError {
    code: number;
    message: string;
}
```

---

## 3.6 错误码

### JS 层错误码

| 错误码 | 常量名 | 消息 | 行号 |
|--------|--------|------|------|
| 202 | `DEVATTEST_ERR_JS_IS_NOT_SYSTEM_APP` | "This api is system api, Please use the system application to call this api" | `native_device_attest.cpp:28-29` |
| 401 | `DEVATTEST_ERR_JS_PARAMETER_ERROR` | "Input paramters wrong" | `native_device_attest.cpp:30` |
| 20000001 | `DEVATTEST_ERR_JS_SYSTEM_SERVICE_EXCEPTION` | "System service exception, please try again or reboot your device" | `native_device_attest.cpp:31-32` |

### 错误处理

```cpp
// interfaces/kit/js/src/native_device_attest.cpp:44-56

static JSIValue GetJsiErrorMessage(int32_t errorCode)
{
    JSIValue error = JSI::CreateObject();
    // ...
    JSI::SetStringProperty(error, "message", GetErrorMessage(errorCode).c_str());
    JSI::SetNumberProperty(error, "code", errorCode);
    return error;
}
```

---

## 3.7 JSI API 使用

### 使用的 JSI 函数

| JSI 函数 | 用途 | 行号 |
|----------|------|------|
| `JSI::SetModuleAPI()` | 注册 JS 可调用函数 | 219-220 |
| `JSI::CreateObject()` | 创建 JS 对象 | 46, 168, 197 |
| `JSI::CreateArray()` | 创建 JS 数组 | 121 |
| `JSI::SetStringProperty()` | 设置字符串属性 | 53, 136 |
| `JSI::SetNumberProperty()` | 设置数值属性 | 54, 114-115 |
| `JSI::SetNamedProperty()` | 设置命名属性 | 134 |
| `JSI::SetPropertyByIndex()` | 设置数组元素 | 130 |
| `JSI::CallFunction()` | 调用 JS 回调 | 67, 78 |
| `JSI::AcquireValue()` | 保留 JS 值引用 | 106-107 |
| `JSI::ReleaseValue()` | 释放 JS 值引用 | 131, 175, 200 |
| `JSI::ReleaseValueList()` | 批量释放 JS 值 | 68, 79, 200 |
| `JSI::ValueIsUndefined()` | 检查 undefined | 61, 73, 84 |
| `JSI::ValueIsArray()` | 检查数组类型 | 122 |

---

## 3.8 编译产物

| 产物 | 类型 | 路径 |
|------|------|------|
| `libkit_device_attest.so` | shared_library | `interfaces/kit/js/BUILD.gn:17` |

### 构建依赖

| 依赖 | 类型 | 说明 |
|------|------|------|
| `devattest_client` | shared_library | Framework 客户端 |
| `hilog_shared` | shared_library | 日志库 |

---

## 相关跳转

- [04_InnerAPI](04_InnerAPI.md) - Inner API 详细说明
- [02_Architecture](02_Architecture.md) - 架构概览
