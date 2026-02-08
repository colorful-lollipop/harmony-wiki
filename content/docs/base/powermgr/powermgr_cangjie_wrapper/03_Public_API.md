# 对外 API (Cangjie API)

> **目的**: 完整列出 powermgr_cangjie_wrapper 的所有公开接口
> **适用范围**: API 使用、接口对接、文档生成
> **最后更新**: 2025-02-06

---

## API 清单表

### BatteryInfo 类

| 属性名 | 类型 | 描述 | 单位 | 同步/异步 | FFI 入口 | 错误码 |
|-------|------|------|------|-----------|---------|-------|
| `batterySoc` | `Int32` | 电池电量百分比 | % | 同步 | `FfiBatteryInfoBatterySOC()` | - |
| `chargingStatus` | `BatteryChargeState` | 充电状态 | - | 同步 | `FfiBatteryInfoGetChargingState()` | 401 |
| `healthStatus` | `BatteryHealthState` | 健康状态 | - | 同步 | `FfiBatteryInfoGetHealthState()` | 401 |
| `pluggedType` | `BatteryPluggedType` | 充电器类型 | - | 同步 | `FfiBatteryInfoGetPluggedType()` | 401 |
| `voltage` | `Int32` | 电池电压 | µV | 同步 | `FfiBatteryInfoGetVoltage()` | - |
| `technology` | `String` | 电池技术类型 | - | 同步 | `FfiBatteryInfoGetTechnology()` | - |
| `batteryTemperature` | `Int32` | 电池温度 | 0.1℃ | 同步 | `FfiBatteryInfoGetBatteryTemperature()` | - |
| `isBatteryPresent` | `Bool` | 电池是否存在 | - | 同步 | `FfiBatteryInfoGetBatteryPresent()` | - |
| `batteryCapacityLevel` | `BatteryCapacityLevel` | 电池容量等级 | - | 同步 | `FfiBatteryInfoGetCapacityLevel()` | 401 |
| `nowCurrent` | `Int32` | 当前电流 | mA | 同步 | `FfiBatteryInfoGetBatteryNowCurrent()` | - |

**证据**: `ohos/battery_info/battery_info.cj:34-170`

---

## 详细 API 说明

### BatteryInfo 类

**包路径**: `ohos.battery_info.BatteryInfo`

**API Level**: 22

**SystemCapability**: `SystemCapability.PowerManager.BatteryManager.Core`

**证据**: `ohos/battery_info/battery_info.cj:30-33, 38-41, 65-68, 79-82, 93-96, 106-109, 122-125, 135-138, 148-151, 162-165`

#### batterySoc - 电池电量

```cangjie
public static prop batterySoc: Int32 {
    get() {
        unsafe { FfiBatteryInfoBatterySOC() }
    }
}
```

**参数**: 无

**返回值**: 电池电量百分比 (0-100)

**异常**: 无

**使用示例**:
```cangjie
let soc = BatteryInfo.batterySoc
println("电池电量: ${soc}%")
```

**证据**: `ohos/battery_info/battery_info.cj:42-46`

#### chargingStatus - 充电状态

```cangjie
public static prop chargingStatus: BatteryChargeState {
    get() {
        let cStatus = unsafe { FfiBatteryInfoGetChargingState() }
        BatteryChargeState.parse(cStatus)
    }
}
```

**参数**: 无

**返回值**: `BatteryChargeState` 枚举值

**异常**: `BusinessException(401, "Parameter error.")` - 当 C 层返回无效值时

**使用示例**:
```cangjie
let status = BatteryInfo.chargingStatus
match (status) {
    case BatteryChargeState.Enabled => println("正在充电")
    case BatteryChargeState.Full => println("已充满")
    case _ => println("未充电或状态未知")
}
```

**证据**: `ohos/battery_info/battery_info.cj:55-60`

#### healthStatus - 健康状态

```cangjie
public static prop healthStatus: BatteryHealthState {
    get() {
        let cStatus = unsafe { FfiBatteryInfoGetHealthState() }
        BatteryHealthState.parse(cStatus)
    }
}
```

**参数**: 无

**返回值**: `BatteryHealthState` 枚举值

**异常**: `BusinessException(401, "Parameter error.")`

**证据**: `ohos/battery_info/battery_info.cj:69-74`

#### pluggedType - 充电器类型

```cangjie
public static prop pluggedType: BatteryPluggedType {
    get() {
        let cPlugged = unsafe { FfiBatteryInfoGetPluggedType() }
        BatteryPluggedType.parse(cPlugged)
    }
}
```

**参数**: 无

**返回值**: `BatteryPluggedType` 枚举值

**异常**: `BusinessException(401, "Parameter error.")`

**证据**: `ohos/battery_info/battery_info.cj:83-88`

#### voltage - 电池电压

```cangjie
public static prop voltage: Int32 {
    get() {
        unsafe { FfiBatteryInfoGetVoltage() }
    }
}
```

**参数**: 无

**返回值**: 电池电压（微伏，µV）

**异常**: 无

**证据**: `ohos/battery_info/battery_info.cj:97-101`

#### technology - 电池技术

```cangjie
public static prop technology: String {
    get() {
        let cStr = unsafe { FfiBatteryInfoGetTechnology() }
        let str = cStr.toString()
        unsafe { LibC.free(cStr) }
        str
    }
}
```

**参数**: 无

**返回值**: 电池技术类型字符串（如 "Li-ion"）

**异常**: 无

**内存管理**: C 层返回的 CString 会被显式释放

**证据**: `ohos/battery_info/battery_info.cj:110-116`

#### batteryTemperature - 电池温度

```cangjie
public static prop batteryTemperature: Int32 {
    get() {
        unsafe { FfiBatteryInfoGetBatteryTemperature() }
    }
}
```

**参数**: 无

**返回值**: 电池温度（0.1 摄氏度）

**异常**: 无

**证据**: `ohos/battery_info/battery_info.cj:126-130`

#### isBatteryPresent - 电池是否存在

```cangjie
public static prop isBatteryPresent: Bool {
    get() {
        unsafe { FfiBatteryInfoGetBatteryPresent() }
    }
}
```

**参数**: 无

**返回值**: `true` 表示电池存在，`false` 表示电池不存在

**异常**: 无

**证据**: `ohos/battery_info/battery_info.cj:139-143`

#### batteryCapacityLevel - 电池容量等级

```cangjie
public static prop batteryCapacityLevel: BatteryCapacityLevel {
    get() {
        let level = unsafe { FfiBatteryInfoGetCapacityLevel() }
        BatteryCapacityLevel.parse(level)
    }
}
```

**参数**: 无

**返回值**: `BatteryCapacityLevel` 枚举值

**异常**: `BusinessException(401, "Parameter error.")`

**证据**: `ohos/battery_info/battery_info.cj:152-157`

#### nowCurrent - 当前电流

```cangjie
public static prop nowCurrent: Int32 {
    get() {
        unsafe { FfiBatteryInfoGetBatteryNowCurrent() }
    }
}
```

**参数**: 无

**返回值**: 当前电流（毫安，mA）

**异常**: 无

**证据**: `ohos/battery_info/battery_info.cj:166-170`

---

## 枚举类型

### BatteryPluggedType - 充电器类型

| 值 | C 层值 | 描述 |
|----|-------|------|
| `UnknownType` | 0 | 未知类型 |
| `Ac` | 1 | AC 充电器 |
| `Usb` | 2 | USB 充电器 |
| `Wireless` | 3 | 无线充电器 |

**解析方法**:
```cangjie
static func parse(value: Int32): BatteryPluggedType {
    match (value) {
        case 0 => UnknownType
        case 1 => Ac
        case 2 => Usb
        case 3 => Wireless
        case _ => throw BusinessException(401, "Parameter error.")
    }
}
```

**证据**: `ohos/battery_info/battery_info.cj:181-228`

### BatteryChargeState - 充电状态

| 值 | C 层值 | 描述 |
|----|-------|------|
| `UnknownChargeState` | 0 | 未知状态 |
| `Enabled` | 1 | 正在充电 |
| `Disabled` | 2 | 未充电 |
| `Full` | 3 | 已充满 |

**解析方法**:
```cangjie
static func parse(value: Int32): BatteryChargeState {
    match (value) {
        case 0 => UnknownChargeState
        case 1 => Enabled
        case 2 => Disabled
        case 3 => Full
        case _ => throw BusinessException(401, "Parameter error.")
    }
}
```

**证据**: `ohos/battery_info/battery_info.cj:238-285`

### BatteryHealthState - 健康状态

| 值 | C 层值 | 描述 |
|----|-------|------|
| `UnknownHealthState` | 0 | 未知状态 |
| `Good` | 1 | 健康 |
| `Overheat` | 2 | 过热 |
| `Overvoltage` | 3 | 过压 |
| `Cold` | 4 | 低温 |
| `Dead` | 5 | 电池失效 |

**解析方法**:
```cangjie
static func parse(value: Int32): BatteryHealthState {
    match (value) {
        case 0 => UnknownHealthState
        case 1 => Good
        case 2 => Overheat
        case 3 => Overvoltage
        case 4 => Cold
        case 5 => Dead
        case _ => throw BusinessException(401, "Parameter error.")
    }
}
```

**证据**: `ohos/battery_info/battery_info.cj:295-362`

### BatteryCapacityLevel - 容量等级

| 值 | C 层值 | 描述 |
|----|-------|------|
| `LevelFull` | 1 | 满电 |
| `LevelHigh` | 2 | 高电量 |
| `LevelNormal` | 3 | 正常电量 |
| `LevelLow` | 4 | 低电量 |
| `LevelWarning` | 5 | 警告低电量 |
| `LevelCritical` | 6 | 严重低电量 |
| `LevelShutdown` | 7 | 即将关机 |

**解析方法**:
```cangjie
static func parse(value: Int32): BatteryCapacityLevel {
    match (value) {
        case 1 => LevelFull
        case 2 => LevelHigh
        case 3 => LevelNormal
        case 4 => LevelLow
        case 5 => LevelWarning
        case 6 => LevelCritical
        case 7 => LevelShutdown
        case _ => throw BusinessException(401, "Parameter error.")
    }
}
```

**证据**: `ohos/battery_info/battery_info.cj:372-449`

---

## FFI 函数清单

### native.cj 中声明的外部函数

| 函数名 | 返回值类型 | 描述 |
|-------|----------|------|
| `FfiBatteryInfoBatterySOC()` | `Int32` | 获取电池电量百分比 |
| `FfiBatteryInfoGetChargingState()` | `Int32` | 获取充电状态 |
| `FfiBatteryInfoGetHealthState()` | `Int32` | 获取健康状态 |
| `FfiBatteryInfoGetPluggedType()` | `Int32` | 获取充电器类型 |
| `FfiBatteryInfoGetVoltage()` | `Int32` | 获取电压 |
| `FfiBatteryInfoGetBatteryNowCurrent()` | `Int32` | 获取电流 |
| `FfiBatteryInfoGetTechnology()` | `CString` | 获取电池技术类型 |
| `FfiBatteryInfoGetBatteryTemperature()` | `Int32` | 获取电池温度 |
| `FfiBatteryInfoGetBatteryPresent()` | `Bool` | 检查电池是否存在 |
| `FfiBatteryInfoGetCapacityLevel()` | `Int32` | 获取电池容量等级 |

**证据**: `ohos/battery_info/native.cj:20-40`

---

## 错误码与异常

### 错误码

| 错误码 | 描述 | 触发条件 |
|-------|------|---------|
| 401 | 参数错误 | C 层返回无效的枚举值 |

### 异常类型

**BusinessException**

- **来源**: `ohos.business_exception.BusinessException`
- **参数**: `(code: Int32, message: String)`
- **用法**: `throw BusinessException(401, "Parameter error.")`

**证据**: `ohos/battery_info/battery_info.cj:225, 282, 359, 446`

---

## 权限与前置条件

### 权限要求

**TODO(需确认)**: 当前 API 层无显式权限检查代码，权限控制可能在外部组件 `battery_manager` 实现。

**建议**: 应用可能需要以下权限（需验证）：
- `ohos.permission.GET_BATTERY_INFO` - 获取电池信息

### 前置条件

- 设备必须支持电池管理功能（`SystemCapability.PowerManager.BatteryManager.Core`）
- 设备必须是标准设备（非轻量级设备）
- 电池驱动必须正常工作

---

## 调用链

### 典型调用链（batterySoc）

```mermaid
graph TD
    A[应用调用] --> B[BatteryInfo.batterySoc]
    B --> C[unsafe { FfiBatteryInfoBatterySOC() }]
    C --> D[FFI: FfiBatteryInfoBatterySOC]
    D --> E[battery_manager:cj_battery_info_ffi]
    E --> F[读取电池硬件]
    F --> G[返回 Int32 值]
    G --> H[返回给应用]
```

### 枚举解析调用链（chargingStatus）

```mermaid
graph TD
    A[应用调用] --> B[BatteryInfo.chargingStatus]
    B --> C[unsafe { FfiBatteryInfoGetChargingState() }]
    C --> D[返回 Int32, 如 1]
    D --> E[BatteryChargeState.parse 1]
    E --> F{match 1}
    F -->|成功| G[返回 BatteryChargeState.Enabled]
    F -->|失败| H[throw BusinessException 401]
    G --> I[返回给应用]
    H --> I
```

### 字符串内存管理调用链（technology）

```mermaid
graph TD
    A[应用调用] --> B[BatteryInfo.technology]
    B --> C[unsafe { FfiBatteryInfoGetTechnology() }]
    C --> D[返回 CString]
    D --> E[cStr.toString]
    E --> F[LibC.free cStr]
    F --> G[返回 String]
    G --> H[返回给应用]
```

---

## 相关跳转

- [项目概览](00_Overview.md) - 项目定位和核心能力
- [架构说明](02_Architecture.md) - 查看组件图和数据流
- [内部 API](04_Internal_API.md) - FFI 接口详解
- [安全风险评审](07_Security_Review.md) - 了解安全注意事项
