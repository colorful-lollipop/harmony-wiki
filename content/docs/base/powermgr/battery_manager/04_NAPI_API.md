# 对外 N-API 接口文档

> **目的**: 详细记录 battery_manager 项目的所有 N-API 接口、参数校验、错误处理和调用链

**适用范围**: JS 应用可调用的所有 N-API 模块

---

## N-API 模块清单

### 1. batteryInfo 模块

**JS 模块名**: `@ohos.batteryInfo`

**模块文件**: `frameworks/napi/src/battery_info.cpp`

**注册函数**: `BatteryInit()` (行 566-600)

**注册宏**: `napi_module_register(&g_module)` (行 619-622)

**证据**: `frameworks/napi/src/battery_info.cpp:606-622`

### 2. battery 模块

**JS 模块名**: `@ohos.battery`

**模块文件**: `frameworks/napi/src/system_battery.cpp`

**注册函数**: `SystemBatteryInit()` (行 261-270)

**注册宏**: `napi_module_register(&g_module)` (行 289-292)

**证据**: `frameworks/napi/src/system_battery.cpp:276-292`

### 3. charger 模块

**JS 模块名**: `@ohos.charger`

**模块文件**: `frameworks/napi/src/charger.cpp`

**注册函数**: `ChargeTypeInit()` (行 82-89)

**注册宏**: `napi_module_register(&g_module)` (行 108-111)

**证据**: `frameworks/napi/src/charger.cpp:95-111`

---

## batteryInfo 模块 API 详细说明

### 属性接口（Getter）

| JS 属性名 | C++ 实现函数 | 返回值类型 | 同步/异步 | 说明 |
|-----------|-------------|-----------|---------|------|
| `batterySOC` | `BatterySOC()` | int32_t | 同步 | 获取电池电量百分比（0-100） |
| `chargingStatus` | `GetChargingState()` | int32_t | 同步 | 获取充电状态（枚举值） |
| `healthStatus` | `GetHealthState()` | int32_t | 同步 | 获取电池健康状态（枚举值） |
| `pluggedType` | `GetPluggedType()` | int32_t | 同步 | 获取充电器类型（枚举值） |
| `voltage` | `GetVoltage()` | int32_t | 同步 | 获取电池电压（mV） |
| `technology` | `GetTechnology()` | string | 同步 | 获取电池技术类型（如 "Li-ion"） |
| `batteryTemperature` | `GetBatteryTemperature()` | int32_t | 同步 | 获取电池温度（0.1℃） |
| `isBatteryPresent` | `GetBatteryPresent()` | boolean | 同步 | 获取电池是否存在 |
| `batteryCapacityLevel` | `GetCapacityLevel()` | int32_t | 同步 | 获取电池电量等级（枚举值） |
| `estimatedRemainingChargeTime` | `GetRemainingChargeTime()` | int64_t | 同步 | 获取预计剩余充电时间（秒） |
| `nowCurrent` | `GetBatteryNowCurrent()` | int32_t | 同步 | 获取当前电流（mA） |
| `remainingEnergy` | `GetBatteryRemainEnergy()` | int32_t | 同步 | 获取剩余电量（mAh） |
| `totalEnergy` | `GetTotalEnergy()` | int32_t | 同步 | 获取总电量（mAh） |

**证据**: `frameworks/napi/src/battery_info.cpp:37-183`

### 函数接口

| JS 方法名 | C++ 实现函数 | 参数 | 返回值 | 同步/异步 | 说明 |
|----------|-------------|------|---------|---------|------|
| `setBatteryConfig` | `SetBatteryConfig()` | sceneName: string, value: string | uint32_t | 同步 | 设置电池配置 |
| `getBatteryConfig` | `GetBatteryConfig()` | sceneName: string | string | 同步 | 获取电池配置 |
| `isBatteryConfigSupported` | `IsBatteryConfigSupported()` | featureName: string | boolean | 同步 | 检查是否支持某配置 |

**证据**: `frameworks/napi/src/battery_info.cpp:185-272`

### 参数解析与校验

#### setBatteryConfig() 参数校验

```cpp
// 参数数量检查
if (argc != INDEX_2) {
    BATTERY_HILOGW(FEATURE_BATT_INFO, "set charge config failed, param is invalid");
    error.ThrowError(env, BatteryError::ERR_PARAM_INVALID);
    return nullptr;
}

// 参数类型检查
if (!NapiUtils::CheckValueType(env, argv[INDEX_0], napi_string)
    || !NapiUtils::CheckValueType(env, argv[INDEX_1], napi_string)) {
    error.ThrowError(env, BatteryError::ERR_PARAM_INVALID);
    return nullptr;
}

// 字符串提取
std::string sceneName = NapiUtils::GetStringFromNapi(env, argv[INDEX_0]);
std::string value = NapiUtils::GetStringFromNapi(env, argv[INDEX_1]);
```

**证据**: `frameworks/napi/src/battery_info.cpp:185-213`

#### getBatteryConfig() 参数校验

```cpp
// 参数数量检查
if (argc != 1 || !NapiUtils::CheckValueType(env, argv[INDEX_0], napi_string)) {
    error.ThrowError(env, BatteryError::ERR_PARAM_INVALID);
    return nullptr;
}

// 字符串提取
std::string sceneName = NapiUtils::GetStringFromNapi(env, argv[INDEX_0]);
```

**证据**: `frameworks/napi/src/battery_info.cpp:215-242`

#### isBatteryConfigSupported() 参数校验

```cpp
// 参数数量检查
if (argc != 1 || !NapiUtils::CheckValueType(env, argv[INDEX_0], napi_string)) {
    error.ThrowError(env, BatteryError::ERR_PARAM_INVALID);
    return nullptr;
}

// 字符串提取
std::string sceneName = NapiUtils::GetStringFromNapi(env, argv[INDEX_0]);
```

**证据**: `frameworks/napi/src/battery_info.cpp:244-272`

### 调用链

#### 同步查询调用链

```
JS 应用
    ↓
N-API: batterySOC (getter)
    ↓
BatterySrvClient::GetInstance().GetCapacity()
    ↓
BatteryService::GetCapacity() [SA 3302]
    ↓
batteryInfo_.GetCapacity()
    ↓
返回 int32_t
```

**证据**:
- N-API: `frameworks/napi/src/battery_info.cpp:37-46`
- 客户端: `interfaces/inner_api/native/include/battery_srv_client.h:38`
- 服务端: `services/native/src/battery_service.cpp:786-795`

#### 配置操作调用链

```
JS 应用
    ↓
N-API: setBatteryConfig(sceneName, value)
    ↓ (参数校验)
NapiUtils::CheckValueType()
    ↓
BatterySrvClient::GetInstance().SetBatteryConfig(sceneName, value)
    ↓ (Binder IPC)
BatteryService::SetBatteryConfig() [SA 3302]
    ↓ (权限检查)
Permission::IsSystem()
    ↓ (如果是系统应用)
BatteryConfig::SetConfigValue()
    ↓
返回 BatteryError
```

**证据**:
- N-API: `frameworks/napi/src/battery_info.cpp:185-213`
- 客户端: `interfaces/inner_api/native/include/battery_srv_client.h:93`
- 服务端: `services/native/src/battery_service.cpp:107-109`

---

## battery 模块 API 详细说明

### 导出方法

| JS 方法名 | C++ 实现函数 | 参数 | 返回值 | 同步/异步 | 说明 |
|----------|-------------|------|---------|---------|------|
| `getStatus` | `GetStatus()` | options: object | void | 异步（Callback） | 获取电池状态，通过 success/fail/complete 回调返回 |

**证据**: `frameworks/napi/src/system_battery.cpp:236-255`

### 参数解析

#### GetStatus() 参数校验

```cpp
// 参数数量检查
if (argc != MAX_ARGC) {
    BATTERY_HILOGW(FEATURE_BATT_INFO, "Lack of parameter, argc: %{public}zu", argc);
    return nullptr;
}

// options 参数类型检查
if (!CheckValueType(env, argv[ARGC_ONE], napi_object)) {
    BATTERY_HILOGW(FEATURE_BATT_INFO, "Check input parameter error");
    return nullptr;
}

// 提取回调函数
successCallBack = GetOptionsFunc(env, options, "success");
failCallBack = GetOptionsFunc(env, options, "fail");
completeCallBack = GetOptionsFunc(env, options, "complete");
```

**证据**: `frameworks/napi/src/system_battery.cpp:245-254`

### 回调函数

| 回调名称 | 参数 | 说明 |
|----------|------|------|
| success | BatteryResponse { level: number, charging: number } | 成功时调用 |
| fail | data: string, code: number | 失败时调用 |
| complete | 无参数 | 完成时调用 |

**证据**: `frameworks/napi/src/system_battery.cpp:35-39`

### 异步处理

```cpp
// 使用 napi_send_event() 发送异步任务
SendEvent(env, asyncInfo.get(), napi_eprio_low, __func__);

// 任务 Lambda 在事件循环中执行
auto task = [env, asyncContext]() mutable {
    if (asyncContext == nullptr) {
        return;
    }
    asyncContext->GetBatteryStats(env);
    delete asyncContext;
};
```

**证据**: `frameworks/napi/src/system_battery.cpp:219-234`

### 调用链

#### 异步获取调用链

```
JS 应用
    ↓
N-API: getStatus(options)
    ↓
创建 SystemBattery 实例
    ↓
CreateCallbackRef(options)
    ↓
SendEvent() (napi_eprio_low)
    ↓
事件循环执行任务
    ↓
GetBatteryStats()
    ↓
BatteryInfo::GetBatteryInfo()
    ↓
BatterySrvClient::GetInstance().GetCapacity()
    ↓
BatteryService::GetCapacity() [SA 3302]
    ↓
返回 BatteryResponse
    ↓
SuccessCallback(response)
    ↓
JS 应用接收成功回调
```

**证据**:
- N-API: `frameworks/napi/src/system_battery.cpp:236-255`
- 异步任务: `frameworks/napi/src/system_battery.cpp:219-234`

---

## charger 模块 API 详细说明

### 导出枚举类

| JS 类名 | C++ 类 | 说明 |
|----------|--------|------|
| `ChargeType` | 枚举类 | 充电类型枚举 |

### 枚举值

| 枚举常量 | 数值 | 说明 |
|----------|------|------|
| `ChargeType.NONE` | 0 | 未知充电类型 |
| `ChargeType.WIRED_NORMAL` | 1 | 有线普通充电 |
| `ChargeType.WIRED_QUICK` | 2 | 有线快充 |
| `ChargeType.WIRED_SUPER_QUICK` | 3 | 有线超级快充 |
| `ChargeType.WIRELESS_NORMAL` | 4 | 无线普通充电 |
| `ChargeType.WIRELESS_QUICK` | 5 | 无线快充 |
| `ChargeType.WIRELESS_SUPER_QUICK` | 6 | 无线超级快充 |

**证据**: `frameworks/napi/src/charger.cpp:41-76`

---

## 错误码与异常封装

### 错误码定义

| 错误码 | 值 | 说明 | N-API 错误消息 |
|--------|-----|------|----------|
| `ERR_OK` | 0 | 成功 | - |
| `ERR_FAILURE` | 1 | 通用失败 | - |
| `ERR_PERMISSION_DENIED` | 201 | 权限被拒绝 | "Permission is denied" |
| `ERR_SYSTEM_API_DENIED` | 202 | 系统 API 权限被拒绝 | "System permission is denied" |
| `ERR_PARAM_INVALID` | 401 | 无效输入参数 | "Invalid input parameter." |
| `ERR_CONNECTION_FAIL` | 5100101 | 连接服务失败 | "Connecting to the service failed." |

**证据**: `interfaces/inner_api/native/include/battery_srv_errors.h:24-27`, `frameworks/napi/src/napi_error.cpp:23-28`

### NAPI 错误封装

```cpp
class NapiError {
public:
    void ThrowError(napi_env& env, BatteryError code) {
        Error(code);
        napi_value error = GetNapiError(env);
        napi_throw(env, error);  // 抛出到 JS 层
    }

private:
    std::map<BatteryError, std::string> errorTable_ = {
        {BatteryError::ERR_CONNECTION_FAIL,   "Connecting to the service failed."},
        {BatteryError::ERR_PERMISSION_DENIED, "Permission is denied"},
        {BatteryError::ERR_SYSTEM_API_DENIED, "System permission is denied"},
        {BatteryError::ERR_PARAM_INVALID,     "Invalid input parameter."}
    };
};
```

**证据**: `frameworks/napi/src/napi_error.cpp:23-65`

### 错误抛出示例

```cpp
// 参数无效
if (/* 参数校验失败 */) {
    error.ThrowError(env, BatteryError::ERR_PARAM_INVALID);
    return nullptr;
}

// 服务调用失败
BatteryError code = g_battClient.SetBatteryConfig(...);
if (code != BatteryError::ERR_OK) {
    error.ThrowError(env, code);
}
```

**证据**: `frameworks/napi/src/battery_info.cpp:192-211, 206-212`

---

## 枚举类型

### BatteryHealthState

| 枚举值 | 数值 | 说明 | 证据 |
|----------|-----|------|------|
| `BatteryHealthState.UNKNOWN` | 0 | 未知健康状态 | `interfaces/inner_api/native/include/battery_info.h:67` |
| `BatteryHealthState.GOOD` | 1 | 健康状态良好 | `interfaces/inner_api/native/include/battery_info.h:72` |
| `BatteryHealthState.OVERHEAT` | 2 | 过热 | `interfaces/inner_api/native/include/battery_info.h:77` |
| `BatteryHealthState.OVERVOLTAGE` | 3 | 过压 | `interfaces/inner_api/native/include/battery_info.h:82` |
| `BatteryHealthState.COLD` | 4 | 过冷 | `interfaces/inner_api/native/include/battery_info.h:87` |
| `BatteryHealthState.DEAD` | 5 | 损坏 | `interfaces/inner_api/native/include/battery_info.h:92` |

**证据**: `interfaces/inner_api/native/include/battery_info.h:63-98`

### BatteryChargeState

| 枚举值 | 数值 | 说明 | 证据 |
|----------|-----|------|------|
| `BatteryChargeState.NONE` | 0 | 未充电 | `interfaces/inner_api/native/include/battery_info.h:37` |
| `BatteryChargeState.ENABLE` | 1 | 充电中 | `interfaces/inner_api/native/include/battery_info.h:42` |
| `BatteryChargeState.DISABLE` | 2 | 未充电 | `interfaces/inner_api/native/include/battery_info.h:47` |
| `BatteryChargeState.FULL` | 3 | 充满 | `interfaces/inner_api/native/include/battery_info.h:52` |

**证据**: `interfaces/inner_api/native/include/battery_info.h:33-58`

### BatteryPluggedType

| 枚举值 | 数值 | 说明 | 证据 |
|----------|-----|------|------|
| `BatteryPluggedType.NONE` | 0 | 无插头 | `interfaces/inner_api/native/include/battery_info.h:107` |
| `BatteryPluggedType.AC` | 1 | AC 充电器 | `interfaces/inner_api/native/include/battery_info.h:112` |
| `BatteryPluggedType.USB` | 2 | USB 充电器 | `interfaces/inner_api/native/include/battery_info.h:117` |
| `BatteryPluggedType.WIRELESS` | 3 | 无线充电器 | `interfaces/inner_api/native/include/battery_info.h:122` |

**证据**: `interfaces/inner_api/native/include/battery_info.h:103-128`

### BatteryCapacityLevel

| 枚举值 | 数值 | 说明 | 证据 |
|----------|-----|------|------|
| `BatteryCapacityLevel.LEVEL_NONE` | 0 | 未知 | `interfaces/inner_api/native/include/battery_info.h:137` |
| `BatteryCapacityLevel.LEVEL_FULL` | 1 | 满电 | `interfaces/inner_api/native/include/battery_info.h:142` |
| `BatteryCapacityLevel.LEVEL_HIGH` | 2 | 高电量 | `interfaces/inner_api/native/include/battery_info.h:147` |
| `BatteryCapacityLevel.LEVEL_NORMAL` | 3 | 正常 | `interfaces/inner_api/native/include/battery_info.h:152` |
| `BatteryCapacityLevel.LEVEL_LOW` | 4 | 低电量 | `interfaces/inner_api/native/include/battery_info.h:157` |
| `BatteryCapacityLevel.LEVEL_WARNING` | 5 | 警告 | `interfaces/inner_api/native/include/battery_info.h:162` |
| `BatteryCapacityLevel.LEVEL_CRITICAL` | 6 | 严重 | `interfaces/inner_api/native/include/battery_info.h:167` |
| `BatteryCapacityLevel.LEVEL_SHUTDOWN` | 7 | 关机电量 | `interfaces/inner_api/native/include/battery_info.h:172` |

**证据**: `interfaces/inner_api/native/include/battery_info.h:133-178`

---

## 关键调用链图

### 同步查询调用链

```mermaid
graph TD
    App[JS 应用] --> NAPI[N-API 层]
    NAPI --> |batterySOC|
    NAPI --> |BatterySrvClient|
    |BatterySrvClient| --> |Connect|
    |Connect| --> SA[BatteryService SA 3302]
    SA --> |GetCapacity|
    |GetCapacity| --> HDI[HDI Battery 接口]
    HDI --> |返回数据|
    |返回数据| --> SA
    SA --> |返回 capacity|
    |返回 capacity| --> |BatterySrvClient|
    |BatterySrvClient| --> NAPI
    NAPI --> App
```

**证据**: `frameworks/napi/src/battery_info.cpp:37-46`, `services/native/src/battery_service.cpp:786-795`

### 异步获取调用链

```mermaid
graph TD
    App[JS 应用] --> NAPI[N-API 层]
    NAPI --> |getStatus|
    NAPI --> |CreateCallbackRef|
    NAPI --> |SendEvent[napi_eprio_low]|
    |SendEvent| --> EventLoop[事件循环]
    EventLoop --> |GetBatteryStats|
    |GetBatteryStats| --> |BatteryInfo|
    |BatteryInfo| --> |BatterySrvClient|
    |BatterySrvClient| --> SA[BatteryService SA 3302]
    SA --> |GetCapacity|
    |GetCapacity| --> |返回数据|
    |返回数据| --> SA
    SA --> |CreateResponse|
    |CreateResponse| --> EventLoop
    EventLoop --> |SuccessCallback|
    |SuccessCallback| --> App
    EventLoop --> |CompleteCallback|
    |CompleteCallback| --> App
```

**证据**: `frameworks/napi/src/system_battery.cpp:219-234, 236-255`

---

## 相关跳转

- [系统架构](03_Architecture.md)
- [内部 API](05_Inner_API.md)

---

**返回**: [导航](SUMMARY.md)
