# N-API 参考手册

## 1. API 清单

### 1.1 数据状态查询

| JS API | 命名空间 | 参数 | 返回值 | 权限 | 同步/异步 |
|--------|----------|------|--------|------|----------|
| isCellularDataEnabled | data | callback?: AsyncCallback\<boolean\> | Promise\<boolean\> | GET_NETWORK_INFO | Async |
| isCellularDataEnabledSync | data | void | boolean | GET_NETWORK_INFO | Sync |
| getCellularDataState | data | callback?: AsyncCallback\<DataConnectState\> | Promise\<DataConnectState\> | 无 | Async |
| getCellularDataFlowType | data | callback?: AsyncCallback\<DataFlowType\> | Promise\<DataFlowType\> | 无 | Async |

### 1.2 数据开关控制

| JS API | 命名空间 | 参数 | 返回值 | 权限 | 同步/异步 |
|--------|----------|------|--------|------|----------|
| enableCellularData | data | callback?: AsyncCallback\<void\> | Promise\<void\> | SET_TELEPHONY_STATE | Async |
| disableCellularData | data | callback?: AsyncCallback\<void\> | Promise\<void\> | SET_TELEPHONY_STATE | Async |

### 1.3 漫游管理

| JS API | 命名空间 | 参数 | 返回值 | 权限 | 同步/异步 |
|--------|----------|------|--------|------|----------|
| isCellularDataRoamingEnabled | data | slotId: number, callback?: AsyncCallback\<boolean\> | Promise\<boolean\> | GET_NETWORK_INFO | Async |
| isCellularDataRoamingEnabledSync | data | slotId: number | boolean | GET_NETWORK_INFO | Sync |
| enableCellularDataRoaming | data | slotId: number, callback?: AsyncCallback\<void\> | Promise\<void\> | SET_TELEPHONY_STATE | Async |
| disableCellularDataRoaming | data | slotId: number, callback?: AsyncCallback\<void\> | Promise\<void\> | SET_TELEPHONY_STATE | Async |

### 1.4 默认卡管理

| JS API | 命名空间 | 参数 | 返回值 | 权限 | 同步/异步 |
|--------|----------|------|--------|------|----------|
| getDefaultCellularDataSlotId | data | callback?: AsyncCallback\<number\> | Promise\<number\> | 无 | Async |
| getDefaultCellularDataSlotIdSync | data | void | number | 无 | Sync |
| getDefaultCellularDataSimId | data | void | number | 无 | Sync |
| setDefaultCellularDataSlotId | data | slotId: number, callback?: AsyncCallback\<void\> | Promise\<void\> | SET_TELEPHONY_STATE | Async |

### 1.5 APN 管理

| JS API | 命名空间 | 参数 | 返回值 | 权限 | 同步/异步 |
|--------|----------|------|--------|------|----------|
| queryApnIds | data | apnInfo: ApnInfo, callback?: AsyncCallback\<Array\<number\>\> | Promise\<Array\<number\>\> | MANAGE_APN_SETTING | Async |
| setPreferredApn | data | apnId: number, callback?: AsyncCallback\<boolean\> | Promise\<boolean\> | MANAGE_APN_SETTING | Async |
| queryAllApns | data | callback?: AsyncCallback\<Array\<ApnInfo\>\> | Promise\<Array\<ApnInfo\>\> | MANAGE_APN_SETTING | Async |
| getActiveApnName | data | callback?: AsyncCallback\<string\> | Promise\<string\> | 无 | Async |

### 1.6 智能开关

| JS API | 命名空间 | 参数 | 返回值 | 权限 | 同步/异步 |
|--------|----------|------|--------|------|----------|
| enableIntelligenceSwitch | data | enable: boolean | number | 无 | Sync |
| getIntelligenceSwitchState | data | void | boolean | 无 | Sync |

---

## 2. 类型定义

### 2.1 DataConnectState

```typescript
export enum DataConnectState {
    DATA_STATE_UNKNOWN = -1,    // 未知状态
    DATA_STATE_DISCONNECTED = 0, // 已断开
    DATA_STATE_CONNECTING = 1,   // 连接中
    DATA_STATE_CONNECTED = 2,   // 已连接
    DATA_STATE_SUSPENDED = 3    // 已暂停
}
```

### 2.2 DataFlowType

```typescript
export enum DataFlowType {
    DATA_FLOW_TYPE_NONE = 0,      // 无流量
    DATA_FLOW_TYPE_DOWN = 1,      // 仅下行
    DATA_FLOW_TYPE_UP = 2,        // 仅上行
    DATA_FLOW_TYPE_UP_DOWN = 3,   // 上下行都有
    DATA_FLOW_TYPE_DORMANT = 4    // 底层链路休眠
}
```

### 2.3 ApnInfo

```typescript
interface ApnInfo {
    apnName: string;
    apn: string;
    mcc: string;
    mnc: string;
    user?: string;
    type?: string;
    proxy?: string;
    mmsproxy?: string;
}
```

---

## 3. 注册入口

### 3.1 模块注册

**文件**: `frameworks/js/napi/src/napi_cellular_data.cpp`
**行号**: 1515-1528

```cpp
static napi_module _cellularDataModule = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = RegistCellularData,
    .nm_modname = "telephony.data",
    .nm_priv = nullptr,
    .reserved = {nullptr},
};

extern "C" __attribute__((constructor)) void RegisterCellularDataModule(void)
{
    napi_module_register(&_cellularDataModule);
}
```

### 3.2 函数注册

**文件**: `frameworks/js/napi/src/napi_cellular_data.cpp`
**行号**: 1481-1512

```cpp
static napi_value RegistCellularData(napi_env env, napi_value exports)
{
    napi_property_descriptor desc[] = {
        DECLARE_NAPI_WRITABLE_FUNCTION("getCellularDataState", GetCellularDataState),
        DECLARE_NAPI_WRITABLE_FUNCTION("isCellularDataEnabled", IsCellularDataEnabled),
        DECLARE_NAPI_WRITABLE_FUNCTION("isCellularDataEnabledSync", IsCellularDataEnabledSync),
        DECLARE_NAPI_WRITABLE_FUNCTION("enableCellularData", EnableCellularData),
        DECLARE_NAPI_WRITABLE_FUNCTION("disableCellularData", DisableCellularData),
        // ... 更多函数
    };
    napi_define_properties(env, exports, sizeof(desc) / sizeof(desc[0]), desc);
    InitEnumDataConnectState(env, exports);
    InitEnumDataFlowType(env, exports);
    return exports;
}
```

---

## 4. 异步工作模式

### 4.1 异步回调模式

```cpp
// 1. 创建 AsyncContext
auto asyncContext = std::make_unique<AsyncContext>();

// 2. 设置回调引用
napi_create_reference(env, parameters[0], DEFAULT_REF_COUNT, &asyncContext->callbackRef);

// 3. 启动异步工作
return NapiUtil::HandleAsyncWork(env, asyncContext.release(), "GetCellularDataState",
    NativeGetCellularDataState, GetCellularDataStateCallback);
```

### 4.2 Promise 模式

```cpp
napi_value result = nullptr;
if (context.callbackRef == nullptr) {
    napi_create_promise(env, &context.deferred, &result);
} else {
    napi_get_undefined(env, &result);
}
// ...
napi_resolve_deferred(env, context.deferred, result);
```

---

## 5. 错误码参考

| 错误码 | 常量 | 说明 |
|--------|------|------|
| 201 | BusinessError.PERMISSION_DENIED | 权限被拒绝 |
| 202 | BusinessError.SYSTEM_API | 非系统应用调用系统 API |
| 401 | BusinessError.PARAMETER_INVALID | 参数错误 |
| 8300001 | ERROR_INVALID_PARAMETER | 无效参数值 |
| 8300002 | ERROR_SERVICE_UNAVAILABLE | 无法连接服务 |
| 8300003 | ERROR_SYSTEM_INTERNAL | 系统内部错误 |
| 8300004 | ERROR_NO_SIM_CARD | 没有 SIM 卡 |
| 8301001 | ERROR_SIM_CARD_NOT_ACTIVATED | SIM 卡未激活 |
| 8300999 | ERROR_UNKNOWN | 未知错误 |

---

## 6. 参数校验

### 6.1 slotId 校验

```cpp
static inline bool IsValidSlotId(int32_t slotId)
{
    return ((slotId >= DEFAULT_SIM_SLOT_ID) && (slotId < SIM_SLOT_COUNT));
}

static inline bool IsValidSlotIdEx(int32_t slotId)
{
    // 支持 VSim 的扩展校验
    return ((slotId >= DEFAULT_SIM_SLOT_ID) && (slotId < SIM_SLOT_COUNT + 1));
}
```

### 6.2 参数匹配

```cpp
static bool MatchCellularDataParameters(napi_env env, const napi_value parameters[], const size_t parameterCount)
{
    switch (parameterCount) {
        case 0:
            return true;
        case 1:
            return NapiUtil::MatchParameters(env, parameters, {napi_function});
        default:
            return false;
    }
}
```

---

## 7. 权限声明

### 7.1 JS API 权限注解

```typescript
/**
 * @permission ohos.permission.GET_NETWORK_INFO
 * @throws { BusinessError } 201 - Permission denied.
 */
function isCellularDataEnabled(callback: AsyncCallback<boolean>): void;

/**
 * @permission ohos.permission.SET_TELEPHONY_STATE
 * @systemapi Hide this for inner system use.
 */
function enableCellularData(callback: AsyncCallback<void>): void;
```

### 7.2 权限常量定义

**文件**: `frameworks/js/napi/src/napi_cellular_data.cpp`

```cpp
static constexpr const char *SET_TELEPHONY_STATE = "ohos.permission.SET_TELEPHONY_STATE";
static constexpr const char *GET_NETWORK_INFO = "ohos.permission.GET_NETWORK_INFO";
static constexpr const char *MANAGE_APN_SETTING = "ohos.permission.MANAGE_APN_SETTING";
```
