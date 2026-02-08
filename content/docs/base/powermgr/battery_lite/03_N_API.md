# N-API 接口文档

> **目的**: 完整说明 battery_lite 对外暴露的 JS API，包括参数、返回值、错误码和绑定位置。  
> **适用范围**: 使用 JS/TS 开发 OpenHarmony 轻量级应用的开发者。  
> **关键结论**: 项目提供 7 个 JS API，均为异步回调模式，通过 `@system.battery` 模块暴露。

---

## API 清单总表

| JS API | 功能 | 参数 | 返回值 | C++ 实现位置 |
|--------|------|------|--------|--------------|
| `battery.BatterySOC()` | 获取电池电量 | `{ success, fail, complete }` | `{ batterySoc: number }` | `battery_module.cpp:44-58` |
| `battery.ChargingStatus()` | 获取充电状态 | `{ success, fail, complete }` | `{ chargingStatus: number }` | `battery_module.cpp:60-74` |
| `battery.HealthStatus()` | 获取健康状态 | `{ success, fail, complete }` | `{ healthStatus: number }` | `battery_module.cpp:76-90` |
| `battery.PluggedType()` | 获取连接类型 | `{ success, fail, complete }` | `{ pluggedType: number }` | `battery_module.cpp:92-106` |
| `battery.Voltage()` | 获取电池电压 | `{ success, fail, complete }` | `{ voltage: number }` | `battery_module.cpp:109-123` |
| `battery.Technology()` | 获取电池技术 | `{ success, fail, complete }` | `{ technology: string }` | `battery_module.cpp:125-139` |
| `battery.Temperature()` | 获取电池温度 | `{ success, fail, complete }` | `{ temperature: number }` | `battery_module.cpp:141-155` |

**证据**: `interfaces/kits/js/@system.battery.d.ts:169-212` 定义了完整的 JS API。

---

## API 详细说明

### 1. battery.BatterySOC()

获取当前设备的电池剩余电量（State of Charge）。

#### 接口定义

```typescript
export interface BatterySocResponse {
    batterySoc: number;  // 电池电量，0-100 百分比
}

export interface GetBatterySOC {
    success?: (data: BatterySocResponse) => void;  // 成功回调
    fail?: (data: string, code: number) => void;  // 失败回调
    complete?: () => void;                          // 完成回调
}

static BatterySOC(options?: GetBatterySOC): void;
```

#### C++ 实现

```cpp
JSIValue BatteryModule::GetBatterySOC(const JSIValue thisVal, const JSIValue *args, uint8_t argsNum)
{
    JSIValue undefValue = JSI::CreateUndefined();
    int32_t batterySoc = 0;
    if ((args == nullptr) || (argsNum == 0) || JSI::ValueIsUndefined(args[0])) {
        return undefValue;  // 参数校验
    }

    batterySoc = GetBatSocImpl();  // 调用 Native 接口
    JSIValue result = JSI::CreateObject();
    JSI::SetNumberProperty(result, "batterySoc", batterySoc);
    SuccessCallBack(thisVal, args[0], result);
    JSI::ReleaseValue(result);
    return undefValue;
}
```

**证据**: `frameworks/js/builtin/src/battery_module.cpp:44-58`。

#### 调用链

```
JS: battery.BatterySOC()
    ↓
JS Framework: BatteryModule::GetBatterySOC()
    ↓
battery_impl.c: GetBatSocImpl()
    ↓
frameworks/native: GetBatSoc()
    ↓
samgr IPC: BatterySocImpl()
    ↓
services: GetSocImpl()
    ↓
返回 battInfo.batSoc
```

#### 使用示例

```javascript
import battery from '@system.battery';

battery.BatterySOC({
    success: (data) => {
        console.log('Battery SoC:', data.batterySoc + '%');
    },
    fail: (data, code) => {
        console.error('Failed to get battery SOC:', data, code);
    },
    complete: () => {
        console.log('BatterySOC request complete');
    }
});
```

---

### 2. battery.ChargingStatus()

获取当前设备的充电状态。

#### 接口定义

```typescript
export interface BatteryChargingStatusResponse {
    chargingStatus: number;  // 充电状态枚举值
}

export interface GetChargingStatus {
    success?: (data: BatteryChargingStatusResponse) => void;
    fail?: (data: string, code: number) => void;
    complete?: () => void;
}

static ChargingStatus(options?: GetChargingStatus): void;
```

#### 返回值枚举

| 值 | 常量 | 含义 |
|----|------|------|
| 0 | `CHARGE_STATE_NONE` | 电池放电中 |
| 1 | `CHARGE_STATE_ENABLE` | 电池充电中 |
| 2 | `CHARGE_STATE_DISABLE` | 电池未充电 |
| 3 | `CHARGE_STATE_FULL` | 电池已充满 |

**证据**: `interfaces/kits/battery_info.h:23-44`。

#### C++ 实现

```cpp
JSIValue BatteryModule::GetChargingStatus(const JSIValue thisVal, const JSIValue *args, uint8_t argsNum)
{
    int32_t chargingStatus = 0;
    // ... 参数校验同上
    chargingStatus = GetChargingStatusImpl();
    JSIValue result = JSI::CreateObject();
    JSI::SetNumberProperty(result, "chargingStatus", chargingStatus);
    SuccessCallBack(thisVal, args[0], result);
    // ...
}
```

**证据**: `frameworks/js/builtin/src/battery_module.cpp:60-74`。

---

### 3. battery.HealthStatus()

获取当前设备的电池健康状态。

#### 接口定义

```typescript
export interface BatteryHealthStatusResponse {
    healthStatus: number;  // 健康状态枚举值
}

export interface GetHealthStatus {
    success?: (data: BatteryHealthStatusResponse) => void;
    fail?: (data: string, code: number) => void;
    complete?: () => void;
}

static HealthStatus(options?: GetHealthStatus): void;
```

#### 返回值枚举

| 值 | 常量 | 含义 |
|----|------|------|
| 0 | `HEALTH_STATE_UNKNOWN` | 未知状态 |
| 1 | `HEALTH_STATE_GOOD` | 状态良好 |
| 2 | `HEALTH_STATE_OVERHEAT` | 过热 |
| 3 | `HEALTH_STATE_OVERVOLTAGE` | 过压 |
| 4 | `HEALTH_STATE_COLD` | 过冷 |
| 5 | `HEALTH_STATE_DEAD` | 电池失效 |

**证据**: `interfaces/kits/battery_info.h:46-75`。

---

### 4. battery.PluggedType()

获取当前连接的充电器类型。

#### 接口定义

```typescript
export interface BatteryGetPluggedTypeResponse {
    pluggedType: number;  // 连接类型枚举值
}

export interface GetPluggedType {
    success?: (data: BatteryGetPluggedTypeResponse) => void;
    fail?: (data: string, code: number) => void;
    complete?: () => void;
}

static PluggedType(options?: GetPluggedType): void;
```

#### 返回值枚举

| 值 | 常量 | 含义 |
|----|------|------|
| 0 | `PLUGGED_TYPE_NONE` | 未连接电源 |
| 1 | `PLUGGED_TYPE_AC` | 交流充电器 |
| 2 | `PLUGGED_TYPE_USB` | USB 充电器 |
| 3 | `PLUGGED_TYPE_WIRELESS` | 无线充电器 |

**证据**: `interfaces/kits/battery_info.h:77-98`。

---

### 5. battery.Voltage()

获取当前设备的电池电压。

#### 接口定义

```typescript
export interface BatteryGetVoltageResponse {
    voltage: number;  // 电压值，单位 mV
}

export interface GetVoltage {
    success?: (data: BatteryGetVoltageResponse) => void;
    fail?: (data: string, code: number) => void;
    complete?: () => void;
}

static Voltage(options?: GetVoltage): void;
```

#### 返回值说明

- 单位：毫伏 (mV)
- 范围：典型值 3000mV - 4200mV（单节锂离子电池）

---

### 6. battery.Technology()

获取当前设备的电池技术/型号。

#### 接口定义

```typescript
export interface BatteryTechnologyResponse {
    technology: string;  // 电池技术型号字符串
}

export interface GetTechnology {
    success?: (data: BatteryTechnologyResponse) => void;
    fail?: (data: string, code: number) => void;
    complete?: () => void;
}

static Technology(options?: GetTechnology): void;
```

#### 返回值示例

- `"Ternary_Lithium"` - 三元锂电池
- `"Lithium_Polymer"` - 锂聚合物电池
- `"Lithium_Ion"` - 锂离子电池

**证据**: `services/src/battery_device.c:26` 显示默认值为 `"Ternary_Lithium"`。

---

### 7. battery.Temperature()

获取当前设备的电池温度。

#### 接口定义

```typescript
export interface BatteryTemperatureResponse {
    temperature: number;  // 温度值，单位 0.1℃
}

export interface GetTemperature {
    success?: (data: BatteryTemperatureResponse) => void;
    fail?: (data: string, code: number) => void;
    complete?: () => void;
}

static Temperature(options?: GetTemperature): void;
```

#### 返回值说明

- 单位：0.1℃
- 范围：典型值 -100 到 600（对应 -10℃ 到 60℃）

---

## 参数校验机制

### 统一参数检查

所有 API 采用相同的参数校验模式：

```cpp
JSIValue BatteryModule::GetBatterySOC(const JSIValue thisVal, const JSIValue *args, uint8_t argsNum)
{
    JSIValue undefValue = JSI::CreateUndefined();
    if ((args == nullptr) || (argsNum == 0) || JSI::ValueIsUndefined(args[0])) {
        return undefValue;  // 参数为空则返回 undefined
    }
    // ... 继续处理
}
```

**证据**: `frameworks/js/builtin/src/battery_module.cpp:44-50`。

### 错误回调触发条件

| 条件 | 触发 | 数据 |
|------|------|------|
| 参数为空 | 返回 `undefined`，不触发回调 | 无 |
| Native 接口返回 `NULL` | 跳过数据设置 | 无数据 |
| 其他错误 | 触发 `fail` 回调 | 错误信息 |

---

## 系统能力要求

使用电池 API 需要声明系统能力：

```json
{
  "sysCap": [
    "SystemCapability.PowerManager.BatteryManager.Lite"
  ]
}
```

**证据**: `interfaces/kits/js/@system.battery.d.ts:17,26,44,58,68,78,88,98,108,118,128,138,148,166` 标注了所有接口的 `@sysCap` 要求。

---

## 导入方式

### ES Module 导入

```javascript
import battery from '@system.battery';

// 使用 battery 对象调用 API
```

### 全局对象访问

```javascript
// 也可以通过全局对象访问
var battery = require('@system.battery');
```

---

## 注意事项

1. **异步回调模式**: 所有 API 均为异步，不会阻塞 JS 线程
2. **参数必须包含回调**: 建议至少提供 `success` 或 `fail` 回调
3. **单位换算**: 电压单位是 mV，温度单位是 0.1℃，需要自行换算
4. **枚举值含义**: 返回值为整数枚举值，需要对照常量定义解读

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [概览](00_Overview.md) | 项目核心能力 |
| [架构说明](02_Architecture.md) | 组件和数据流 |
| [内部 API](04_Inner_API.md) | Native API 详细说明 |
| [安全评审](06_Security_Review.md) | API 安全考量 |
