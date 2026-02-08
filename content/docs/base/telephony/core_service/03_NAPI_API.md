# N-API 接口（JS API）

## 目的

本文档详细描述 `telephony_core_service` 提供的 N-API 接口，包括 JS API 清单、参数定义、权限要求和调用链。

---

## 模块概述

| 模块 | 输出产物 | 注册函数 | 命名空间 |
|------|----------|----------|----------|
| SIM | `sim.z.so` | `RegisterSimCardModule()` | `@ohos.telephony.sim` |
| Radio | `radio.z.so` | `RegisterRadioNetworkModule()` | `@ohos.telephony.radio` |
| eSIM | `esim.z.so` | `RegisterEsimCardModule()` | `@ohos.telephony.esim` |
| VCard | `vcard.z.so` | `RegisterVCardModule()` | - |

---

## SIM 模块 API

### 注册信息

**代码证据** (`frameworks/js/sim/src/napi_sim.cpp:3414-3421`):

```cpp
static napi_module _simModule = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = NapiSim::Register,
    .nm_modname = "telephony.sim",
    .nm_priv = nullptr,
    .reserved = {0},
};

extern "C" __attribute__((constructor)) void RegisterSimCardModule(void)
{
    napi_module_register(&_simModule);
}
```

### API 清单

| JS API | C++ 实现 | 参数 | 权限 | 同步/异步 |
|--------|----------|------|------|-----------|
| `getSimState(slotId)` | `GetSimState()` | slotId: number | - | 异步 |
| `hasSimCard(slotId)` | `HasSimCard()` | slotId: number | - | 异步 |
| `getSimIccId(slotId)` | `GetSimIccId()` | slotId: number | GET_TELEPHONY_STATE | 异步 |
| `getIMSI(slotId)` | `GetIMSI()` | slotId: number | GET_TELEPHONY_STATE | 异步 |
| `getSimOperatorNumeric(slotId)` | `GetSimOperatorNumeric()` | slotId: number | - | 异步 |
| `getISOCountryCodeForSim(slotId)` | `GetISOCountryCodeForSim()` | slotId: number | - | 异步 |
| `getSimSpn(slotId)` | `GetSimSpn()` | slotId: number | - | 异步 |
| `getSimGid1(slotId)` | `GetSimGid1()` | slotId: number | GET_TELEPHONY_STATE | 异步 |
| `getSimTelephoneNumber(slotId)` | `GetSimTelephoneNumber()` | slotId: number | GET_PHONE_NUMBERS | 异步 |
| `getVoiceMailIdentifier(slotId)` | `GetVoiceMailIdentifier()` | slotId: number | GET_TELEPHONY_STATE | 异步 |
| `getVoiceMailNumber(slotId)` | `GetVoiceMailNumber()` | slotId: number | GET_TELEPHONY_STATE | 异步 |
| `getCardType(slotId)` | `GetCardType()` | slotId: number | - | 异步 |
| `getSimAccountInfo(slotId)` | `GetSimAccountInfo()` | slotId: number | - | 异步 |
| `isSimActive(slotId)` | `IsSimActive()` | slotId: number | - | 异步 |
| `setSimActive(slotId, enable)` | `SetActiveSim()` | slotId, enable | - | 异步 |
| `getDefaultVoiceSlotId()` | `GetDefaultVoiceSlotId()` | - | - | 异步 |
| `setDefaultVoiceSlotId(slotId)` | `SetDefaultVoiceSlotId()` | slotId | - | 异步 |
| `getPrimarySlotId()` | `GetPrimarySlotId()` | - | - | 异步 |
| `setPrimarySlotId(slotId)` | `SetPrimarySlotId()` | slotId | - | 异步 |
| `getShowNumber(slotId)` | `GetShowNumber()` | slotId | GET_TELEPHONY_STATE | 异步 |
| `setShowNumber(slotId, number)` | `SetShowNumber()` | slotId, number | SET_TELEPHONY_STATE | 异步 |
| `getShowName(slotId)` | `GetShowName()` | slotId | GET_TELEPHONY_STATE | 异步 |
| `setShowName(slotId, name)` | `SetShowName()` | slotId, name | SET_TELEPHONY_STATE | 异步 |
| `unlockPin(slotId, pin)` | `UnlockPin()` | slotId, pin | - | 异步 |
| `unlockPuk(slotId, newPin, puk)` | `UnlockPuk()` | slotId, newPin, puk | - | 异步 |
| `alterPin(slotId, newPin, oldPin)` | `AlterPin()` | slotId, newPin, oldPin | - | 异步 |
| `getLockState(slotId, lockType)` | `GetLockState()` | slotId, lockType | - | 异步 |
| `setLockState(slotId, options)` | `SetLockState()` | slotId, LockInfo | - | 异步 |
| `queryIccDiallingNumbers(slotId, type)` | `QueryIccDiallingNumbers()` | slotId, type | READ_CONTACTS | 异步 |
| `addIccDiallingNumbers(...)` | `AddIccDiallingNumbers()` | ... | WRITE_CONTACTS | 异步 |
| `delIccDiallingNumbers(...)` | `DelIccDiallingNumbers()` | ... | WRITE_CONTACTS | 异步 |
| `updateIccDiallingNumbers(...)` | `UpdateIccDiallingNumbers()` | ... | WRITE_CONTACTS | 异步 |
| `sendEnvelopeCmd(slotId, cmd)` | `SendEnvelopeCmd()` | slotId, cmd | - | 异步 |
| `sendTerminalResponseCmd(slotId, cmd)` | `SendTerminalResponseCmd()` | slotId, cmd | - | 异步 |
| `simAuthentication(slotId, authType, authData)` | `SimAuthentication()` | ... | - | 异步 |

### 参数校验

**代码证据** (`frameworks/js/sim/src/napi_sim.cpp:52-66`):

```cpp
static inline bool IsValidSlotId(int32_t slotId)
{
    return ((slotId >= DEFAULT_SIM_SLOT_ID) && (slotId < SIM_SLOT_COUNT));
}

static inline bool IsValidSlotIdEx(int32_t slotId)
{
    // One more slot for VSim.
    return ((slotId >= DEFAULT_SIM_SLOT_ID) && (slotId < SIM_SLOT_COUNT + 1));
}
```

### 调用链示例

```
JS: getSimState(slotId)
└── napi_sim.cpp: NapiSim::GetSimState()
    └── NapiCreateAsyncWork<SimStateContext, ...>
        └── NativeGetSimState (execute)
            └── CoreServiceClient::GetSimState()
                └── IPC -> CoreService::GetSimState()
                    └── SimManager::GetSimState()
                        └── 返回缓存状态或查询 RIL
        └── GetSimStateCallback (complete)
            └── napi_resolve_deferred / napi_call_function
```

---

## Radio 模块 API

### 注册信息

**代码证据** (`frameworks/js/network_search/src/napi_radio.cpp:3731-3742`):

```cpp
static napi_module _radioModule = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = NapiRadio::Register,
    .nm_modname = "telephony.radio",
    .nm_priv = nullptr,
    .reserved = { 0 },
};

extern "C" __attribute__((constructor)) void RegisterRadioNetworkModule(void)
{
    napi_module_register(&_radioModule);
}
```

### API 清单

| JS API | C++ 实现 | 参数 | 权限 | 同步/异步 |
|--------|----------|------|------|-----------|
| `getRadioTech(slotId)` | `GetRadioTech()` | slotId | GET_NETWORK_INFO | 异步 |
| `getSignalInformation(slotId)` | `GetSignalInformation()` | slotId | - | 异步 |
| `getNetworkState(slotId)` | `GetNetworkState()` | slotId | GET_NETWORK_INFO | 异步 |
| `getNetworkSelectionMode(slotId)` | `GetNetworkSelectionMode()` | slotId | - | 异步 |
| `setNetworkSelectionMode(options)` | `SetNetworkSelectionMode()` | options | SET_TELEPHONY_STATE | 异步 |
| `getNetworkSearchInformation(slotId)` | `GetNetworkSearchInformation()` | slotId | GET_TELEPHONY_STATE | 异步 |
| `isRadioOn(slotId?)` | `IsRadioOn()` | slotId? | GET_NETWORK_INFO | 异步 |
| `turnOnRadio(slotId?)` | `TurnOnRadio()` | slotId? | SET_TELEPHONY_STATE | 异步 |
| `turnOffRadio(slotId?)` | `TurnOffRadio()` | slotId? | SET_TELEPHONY_STATE | 异步 |
| `getISOCountryCodeForNetwork(slotId)` | `GetISOCountryCodeForNetwork()` | slotId | - | 异步 |
| `getOperatorName(slotId)` | `GetOperatorName()` | slotId | - | 异步 |
| `getPreferredNetwork(slotId)` | `GetPreferredNetwork()` | slotId | GET_TELEPHONY_STATE | 异步 |
| `setPreferredNetwork(slotId, mode)` | `SetPreferredNetwork()` | slotId, mode | SET_TELEPHONY_STATE | 异步 |
| `getCellInformation(slotId)` | `GetCellInformation()` | slotId | LOCATION + APPROXIMATELY_LOCATION | 异步 |
| `sendUpdateCellLocationRequest(slotId)` | `SendUpdateCellLocationRequest()` | slotId | LOCATION + APPROXIMATELY_LOCATION | 异步 |
| `getIMEI(slotId)` | `GetImei()` | slotId | GET_TELEPHONY_STATE | 异步 |
| `getMEID(slotId)` | `GetMeid()` | slotId | GET_TELEPHONY_STATE | 异步 |
| `getUniqueDeviceId(slotId)` | `GetUniqueDeviceId()` | slotId | GET_TELEPHONY_STATE | 异步 |
| `isNrSupported()` | `IsNrSupported()` | - | - | 同步 |
| `getNrOptionMode(slotId)` | `GetNrOptionMode()` | slotId | GET_TELEPHONY_STATE | 异步 |
| `setNrOptionMode(slotId, mode)` | `SetNrOptionMode()` | slotId, mode | SET_TELEPHONY_STATE | 异步 |
| `getImsRegInfo(slotId, imsType)` | `GetImsRegStatus()` | slotId, imsType | GET_TELEPHONY_STATE | 异步 |
| `on('imsRegStateChange', ...)` | `RegisterImsRegInfoCallback()` | callback | GET_TELEPHONY_STATE | 事件 |
| `off('imsRegStateChange', ...)` | `UnregisterImsRegInfoCallback()` | callback | GET_TELEPHONY_STATE | 事件 |

### 调用链示例

```
JS: getNetworkState(slotId)
└── napi_radio.cpp: NapiRadio::GetNetworkState()
    └── NativeGetNetworkState (execute)
        └── CoreServiceClient::GetNetworkState()
            └── IPC -> CoreService::GetNetworkState()
                └── NetworkSearchManager::GetNetworkState()
                    └── 返回 NetworkState 对象
    └── GetNetworkStateCallback (complete)
        └── 构造 NetworkState JS 对象
```

---

## eSIM 模块 API

### 注册信息

**代码证据** (`frameworks/js/esim/src/napi_esim.cpp:2083-2091`):

```cpp
static napi_module _esimModule = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = nullptr,
    .nm_register_func = NapiESim::Register,
    .nm_modname = "telephony.esim",
    .nm_priv = nullptr,
    .reserved = {0},
};

extern "C" __attribute__((constructor)) void RegisterEsimCardModule(void)
{
    napi_module_register(&_esimModule);
}
```

### API 清单

| JS API | 参数 | 权限 | 说明 |
|--------|------|------|------|
| `getEid(slotId)` | slotId | GET_TELEPHONY_ESIM_STATE | 获取 EID |
| `getEuiccInfo(slotId)` | slotId | GET_TELEPHONY_ESIM_STATE | 获取 eUICC 信息 |
| `getEuiccProfileInfoList(slotId)` | slotId | GET_TELEPHONY_ESIM_STATE | 获取 Profile 列表 |
| `downloadProfile(slotId, config)` | slotId, DownloadableProfile | SET_TELEPHONY_ESIM_STATE | 下载 Profile |
| `switchToProfile(slotId, iccId, force)` | ... | SET_TELEPHONY_ESIM_STATE_OPEN | 切换 Profile |
| `deleteProfile(slotId, iccId)` | slotId, iccId | SET_TELEPHONY_ESIM_STATE | 删除 Profile |
| `resetMemory(slotId, options)` | slotId, ResetOption | SET_TELEPHONY_ESIM_STATE | 重置内存 |
| `setProfileNickname(slotId, iccId, name)` | ... | SET_TELEPHONY_ESIM_STATE_OPEN | 设置昵称 |
| `getDefaultSmdpAddress(slotId)` | slotId | GET_TELEPHONY_ESIM_STATE | 获取默认 SM-DP+ 地址 |
| `setDefaultSmdpAddress(slotId, addr)` | ... | SET_TELEPHONY_ESIM_STATE | 设置默认 SM-DP+ 地址 |
| `cancelSession(slotId, token, type)` | ... | SET_TELEPHONY_ESIM_STATE | 取消会话 |
| `startOsu(slotId)` | slotId | SET_TELEPHONY_ESIM_STATE | 开始 OSU |

---

## 权限常量定义

**代码证据** (`utils/common/include/telephony_permission.h:27-125`):

```cpp
namespace Permission {
static constexpr const char *GET_TELEPHONY_STATE = "ohos.permission.GET_TELEPHONY_STATE";
static constexpr const char *SET_TELEPHONY_STATE = "ohos.permission.SET_TELEPHONY_STATE";
static constexpr const char *GET_NETWORK_INFO = "ohos.permission.GET_NETWORK_INFO";
static constexpr const char *SET_NETWORK_INFO = "ohos.permission.SET_NETWORK_INFO";
static constexpr const char *GET_PHONE_NUMBERS = "ohos.permission.GET_PHONE_NUMBERS";
static constexpr const char *CELL_LOCATION = "ohos.permission.LOCATION";
static constexpr const char *READ_CONTACTS = "ohos.permission.READ_CONTACTS";
static constexpr const char *WRITE_CONTACTS = "ohos.permission.WRITE_CONTACTS";
static constexpr const char *GET_TELEPHONY_ESIM_STATE = "ohos.permission.GET_TELEPHONY_ESIM_STATE";
static constexpr const char *SET_TELEPHONY_ESIM_STATE = "ohos.permission.SET_TELEPHONY_ESIM_STATE";
// ... 更多权限
}
```

---

## 错误码处理

### N-API 错误转换

**代码证据** (`frameworks/js/napi/napi_util.cpp`):

```cpp
JsError ConverErrorMessageForJs(NapiError error) {
    switch (error.errorCode) {
        case ERROR_SLOT_ID_INVALID:
            return JsError{ERROR_SLOT_ID_INVALID, "Invalid slot id"};
        case ERROR_PARAMETER_TYPE_INVALID:
            return JsError{ERROR_PARAMETER_TYPE_INVALID, "Parameter type invalid"};
        // ...
    }
}
```

### 常见错误码

| 错误码 | 值 | 说明 |
|--------|-----|------|
| `ERROR_SLOT_ID_INVALID` | 10001 | 无效的卡槽 ID |
| `ERROR_PARAMETER_TYPE_INVALID` | 10002 | 参数类型无效 |
| `ERROR_PARAMETER_VALUE_INVALID` | 10003 | 参数值无效 |
| `ERROR_PERMISSION_DENIED` | 10004 | 权限被拒绝 |

## 相关链接

- [架构设计](./02_Architecture.md)
- [内部 API](./04_Inner_API.md)
- [安全风险](./06_Security.md)
