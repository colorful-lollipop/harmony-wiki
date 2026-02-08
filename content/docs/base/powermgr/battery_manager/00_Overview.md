# Battery Manager 项目概览

> **目的**: 提供电池管理组件的高层概览，帮助理解项目定位和核心功能

**适用范围**: battery_manager 项目的整体架构、核心能力、运行环境

---

## 项目定位

### 在系统中的角色

Battery Manager 是 OpenHarmony **电源管理子系统** 的核心组件之一，负责：

1. **电池状态监控** - 实时监控电池电量、温度、健康状态等
2. **充电管理** - 监控充放电状态、充电器类型
3. **状态上报** - 向系统和其他组件上报电池变化事件
4. **关机充电** - 提供关机状态下的充电界面和提示

### 相关组件

- **上游依赖**: `drivers_interface_battery` (HDI 电池驱动)
- **下游服务**: `powermgr_power_manager`、`powermgr_battery_statistics`
- **协同组件**: `common_event_service`（事件通知）、`distributed_notification_service`（低电量通知）

---

## 核心能力

### 1. 电池信息查询

提供以下电池信息查询接口：

| 信息类型 | 描述 | 数据来源 |
|---------|------|---------|
| 电量百分比 | 当前电池电量（0-100%） | HDI Battery Interface |
| 充电状态 | 当前充电状态（充电中/未充电/已充满） | HDI Battery Interface |
| 健康状态 | 电池健康程度（良好/过热/过压/过冷/损坏） | HDI Battery Interface |
| 充电器类型 | 连接的充电器类型（AC/USB/无线） | HDI Battery Interface |
| 电压 | 电池电压（mV） | HDI Battery Interface |
| 温度 | 电池温度（0.1℃） | HDI Battery Interface |
| 技术类型 | 电池技术（如 Li-ion） | HDI Battery Interface |
| 当前电流 | 实时电流（mA） | HDI Battery Interface |
| 平均电流 | 平均电流（mA） | HDI Battery Interface |
| 剩余电量 | 剩余电量（mAh） | HDI Battery Interface |
| 总电量 | 总电量（mAh） | HDI Battery Interface |
| 电量等级 | 电量等级（满电/高/正常/低/警告/严重/关机） | 配置文件 |
| 剩余充电时间 | 预计充电时间（秒） | 算法计算 |
| 电池存在 | 电池是否存在 | HDI Battery Interface |

**证据**:
- 枚举定义: `interfaces/inner_api/native/include/battery_info.h:16-474`
- 客户端接口: `interfaces/inner_api/native/include/battery_srv_client.h:32-101`
- 服务端接口: `services/zidl/IBatterySrv.idl:17-35`

### 2. 充放电状态管理

| 能力 | 描述 |
|------|------|
| 充电状态变化检测 | 监听 HDI 回调，检测充放电状态变化 |
| 充电器插拔检测 | 检测充电器连接/断开 |
| 充电类型识别 | 识别有线普通/快充/超级快充、无线普通/快充/超级快充 |
| 低电量警告 | 在电量低于阈值时触发通知 |
| 极低电量关机 | 在电量低于关机阈值时触发关机（可选） |

**证据**:
- 枚举定义: `interfaces/inner_api/native/include/battery_info.h:183-213` (ChargeType)
- 服务类: `services/native/include/battery_service.h:49-92`

### 3. 关机充电（可选功能）

| 能力 | 描述 |
|------|------|
| 关机充电界面 | 显示充电动画、电量百分比、低电量提示 |
| 充电动画 | 支持自定义充电动画配置 |
| 设备控制 | 支持背光、LED、振动控制 |
| 图形引擎 | 支持 Framebuffer/DRM 驱动 |

**证据**:
- 充电动画配置: `charger/sa_profile/animation.json`
- 设备驱动: `charger/src/dev/drm_driver.cpp`, `charger/src/dev/fbdev_driver.cpp`
- 特性开关: `batterymgr.gni:16-23`

### 4. 配置管理

| 配置类型 | 描述 |
|---------|------|
| 设置电池配置 | 通过 `SetBatteryConfig()` 设置场景化配置 |
| 获取电池配置 | 通过 `GetBatteryConfig()` 获取当前配置 |
| 检查配置支持 | 通过 `IsBatteryConfigSupported()` 检查是否支持某配置 |

**证据**:
- N-API 实现: `frameworks/napi/src/battery_info.cpp:185-272`
- IDL 接口: `services/zidl/IBatterySrv.idl:32-34`

---

## 运行环境

### 系统要求

- **系统类型**: OpenHarmony 标准系统 (`adapted_system_type: ["standard"]`)
- **子系统**: powermgr
- **系统能力**: `SystemCapability.PowerManager.BatteryManager.Core`

**证据**: `bundle.json:16-30`

### 资源占用

- **ROM**: 1024KB
- **RAM**: 2048KB

**证据**: `bundle.json:31-32`

### 进程要求

- **运行进程**: powermgr
- **服务库**: `libbatteryservice.z.so`
- **启动方式**: `run-on-create: true`

**证据**: `sa_profile/3302.json:1-12`

---

## 关键概念

### 1. HDI (Hardware Driver Interface)

HDI 是 OpenHarmony 的硬件驱动接口标准，Battery Manager 通过 HDI 与底层电池驱动交互。

**相关接口**:
- `OHOS::HDI::Battery::V2_0::IBatteryInterface` - 电池驱动接口
- `OHOS::HDI::Battery::V2_0::IBatteryCallback` - 电池回调接口

**证据**: `services/native/include/battery_service.h:33-48`

### 2. ZIDL (Z Interface Definition Language)

ZIDL 是 OpenHarmony 的跨进程接口定义语言，用于生成 IPC Skeleton/Proxy。

**接口文件**: `services/zidl/IBatterySrv.idl`

**证据**: `services/zidl/IBatterySrv.idl:17-35`

### 3. System Ability (SA)

SA 是 OpenHarmony 的系统服务框架，Battery Service 注册为 SA 3302。

**服务 ID**: `POWER_MANAGER_BATT_SERVICE_ID = 3302`

**证据**:
- SA 声明: `services/native/include/battery_service.h:51`
- 配置文件: `sa_profile/3302.json:5`

### 4. CommonEvent

CommonEvent 是 OpenHarmony 的公共事件机制，Battery Manager 通过 CommonEvent 上报电池状态变化。

**事件名称**: `usual.event.BATTERY_CHANGED_INNER`

**证据**: `interfaces/inner_api/native/include/battery_info.h:450`

---

## 模块边界

### 输入边界

| 输入来源 | 说明 |
|-----------|------|
| HDI Battery Interface | 来自底层驱动的硬件事件 |
| IPC 客户端调用 | 来自其他进程的服务请求 |
| 配置文件 | 启动时加载的配置 |
| CommonEvent | 来自其他模块的事件订阅 |

### 输出边界

| 输出对象 | 说明 |
|-----------|------|
| N-API JS 接口 | 供应用层调用的 JS API |
| C API | 供 Native 模块调用的 C 接口 |
| CJ FFI | 供 ArkUI-X 模块调用的 FFI 接口 |
| CommonEvent | 向系统上报的电池变化事件 |
| HiSysEvent | 向系统记录的统计事件 |
| 通知服务 | 低电量通知 |

**证据**:
- N-API 模块: `frameworks/napi/BUILD.gn:16-22`
- C API: `frameworks/capi/BUILD.gn:24-26`
- CJ FFI: `frameworks/cj/BUILD.gn:...`
- HiSysEvent: `batterymgr.yaml:16-31`

---

## 设计模式

### 1. 客户端-服务端模式

```
应用层 (N-API/C-API/CJ FFI)
    ↓
IPC (ZIDL)
    ↓
服务层 (BatteryService SA 3302)
    ↓
HDI (Battery Interface)
    ↓
硬件驱动
```

**证据**:
- 客户端: `interfaces/inner_api/native/include/battery_srv_client.h`
- 服务端: `services/native/include/battery_service.h`
- IPC 接口: `services/zidl/IBatterySrv.idl`

### 2. 单例模式

`BatterySrvClient` 使用 `DelayedRefSingleton` 确保全局唯一实例。

**证据**: `interfaces/inner_api/native/include/battery_srv_client.h:29`

### 3. 回调模式

`BatteryService` 通过注册 HDI 回调监听底层电池事件变化。

**证据**: `services/native/include/battery_service.h:26` (BatteryCallback)

---

## 特性开关

### 配置开关（从 bundle.json）

| 特性开关 | 默认值 | 描述 |
|---------|---------|------|
| `battery_manager_feature_enable_charger` | false | 启用充电模块 |
| `battery_manager_feature_enable_charging_sound` | false | 启用充电提示音 |
| `battery_manager_feature_enable_wireless_charge` | false | 启用无线充电支持 |
| `battery_manager_feature_set_low_capacity_threshold` | false | 启用低电量关机阈值设置 |
| `battery_manager_feature_support_notification` | false | 支持电池通知 |

**证据**: `bundle.json:21-27`

### 条件编译宏

| 宏名称 | 默认值 | 描述 |
|--------|---------|------|
| `HAS_HIVIEWDFX_HISYSEVENT_PART` | false | 是否有 hiviewdfx_hisysevent 模块 |
| `HAS_SENSORS_MISCDEVICE_PART` | false | 是否有 sensors_miscdevice 模块 |
| `HAS_DRIVERS_INTERFACE_DISPLAY_PART` | false | 是否有 drivers_interface_display 模块 |
| `HAS_DRIVERS_INTERFACE_LIGHT_PART` | false | 是否有 drivers_interface_light 模块 |
| `HAS_GRAPHIC_SURFACE_PART` | false | 是否有 graphic_surface 模块 |
| `HAS_UI_LITE_PART` | false | 是否有 ui_lite 模块 |
| `HAS_BATTERY_CONFIG_POLICY_PART` | false | 是否有 customization_config_policy 模块 |

**证据**: `batterymgr.gni:26-79`

---

## 相关跳转

- [项目定位详情](01_Project_Position.md)
- [目录结构](02_Directory_Structure.md)
- [系统架构](03_Architecture.md)
- [N-API 文档](04_NAPI_API.md)

---

**返回**: [导航](SUMMARY.md)
