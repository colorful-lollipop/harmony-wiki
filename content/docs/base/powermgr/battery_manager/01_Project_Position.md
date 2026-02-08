# 项目定位与边界

> **目的**: 明确 battery_manager 项目的功能边界、核心能力和适用场景

**适用范围**: 功能范围、技术边界、依赖关系

---

## 功能边界

### 核心职责

| 职责 | 说明 | 证据 |
|------|------|------|
| 电池状态监控 | 监控电池电量、电压、温度、健康状态等信息 | `services/native/include/battery_service.h:49-92` |
| 充放电状态管理 | 检测和管理充放电状态变化 | `services/native/include/battery_service.h:72-91` |
| 状态上报 | 通过 CommonEvent 上报电池状态变化 | `services/native/src/battery_notify.cpp` |
| 配置管理 | 提供电池配置的设置和查询接口 | `services/native/include/battery_service.h:89-91` |
| 关机充电（可选）| 提供关机状态下的充电界面 | `charger/sa_profile/animation.json` |

### 不包含的功能

| 功能 | 说明 | 理由 |
|------|------|------|
| 充电控制 | 不控制充电器的开关/电流调节 | 由底层电源管理模块负责 |
| 电池保护 | 不直接进行电池保护动作 | 由底层驱动和电源管理负责 |
| 电池校准 | 不提供电池容量校准功能 | 不在当前功能范围内 |
| 多电池支持 | 当前设计支持单电池 | IDL 接口未提供多电池区分 |

---

## 技术边界

### 架构分层

```
┌─────────────────────────────────────────────┐
│          应用层                 │
│  - N-API 应用 (JS)             │
│  - C API 应用 (Native)         │
│  - CJ FFI 应用 (ArkUI-X)       │
└──────────────┬──────────────────────────┘
               │
               ↓ IPC (ZIDL)
┌──────────────┴──────────────────────────┐
│          服务层 (SA 3302)      │
│  - BatteryService                 │
│  - BatteryNotify                 │
│  - BatteryLight                  │
└──────────────┬──────────────────────────┘
               │
               ↓ HDI
┌──────────────┴──────────────────────────┐
│          驱动层 (HDI)          │
│  - IBatteryInterface             │
│  - IBatteryCallback             │
└──────────────┬──────────────────────────┘
               │
               ↓
┌──────────────┴──────────────────────────┐
│          硬件层                  │
│  - 电池芯片                    │
│  - 充电芯片                    │
└───────────────────────────────────────┘
```

**证据**:
- IPC 层: `services/zidl/IBatterySrv.idl`
- HDI 层: `services/native/include/battery_service.h:33-48`
- 客户端: `interfaces/inner_api/native/include/battery_srv_client.h:29`

### 模块职责划分

| 层级 | 模块 | 职责 |
|------|------|------|
| 应用层 | N-API (batteryInfo, battery, charger) | 提供 JS/Native 调用接口 |
| 应用层 | CJ FFI (battery_info_ffi) | 提供 ArkUI-X FFI 接口 |
| 应用层 | C API (ohbattery_info) | 提供 C 语言调用接口 |
| 客户端层 | BatterySrvClient | IPC 客户端代理，连接 SA 3302 |
| 服务端层 | BatteryService | 实现 SA 3302，处理业务逻辑 |
| IPC 层 | ZIDL (IBatterySrv) | 生成 Skeleton/Proxy，定义跨进程接口 |
| 驱动适配层 | BatteryCallback, HdiServiceStatusListener | 适配 HDI 接口 |
| 通知层 | BatteryNotify | CommonEvent 通知管理 |
| 充电层 | charger（可选）| 关机充电界面和动画 |

**证据**:
- 模块声明: `bundle.json:77-95` (inner_kits)

---

## 核心能力

### 电池信息查询

**能力**: 提供完整的电池信息查询接口

| 接口 | 类型 | 返回值 | 证据 |
|------|------|--------|------|
| GetCapacity() | 同步 | int32_t (电量百分比) | `interfaces/inner_api/native/include/battery_srv_client.h:38` |
| GetChargingStatus() | 同步 | BatteryChargeState (充电状态) | `interfaces/inner_api/native/include/battery_srv_client.h:43` |
| GetHealthStatus() | 同步 | BatteryHealthState (健康状态) | `interfaces/inner_api/native/include/battery_srv_client.h:48` |
| GetPluggedType() | 同步 | BatteryPluggedType (充电器类型) | `interfaces/inner_api/native/include/battery_srv_client.h:53` |
| GetVoltage() | 同步 | int32_t (电压 mV) | `interfaces/inner_api/native/include/battery_srv_client.h:57` |
| GetPresent() | 同步 | bool (电池存在) | `interfaces/inner_api/native/include/battery_srv_client.h:61` |
| GetTechnology() | 同步 | string (技术类型) | `interfaces/inner_api/native/include/battery_srv_client.h:65` |
| GetBatteryTemperature() | 同步 | int32_t (温度 0.1℃) | `interfaces/inner_api/native/include/battery_srv_client.h:69` |
| GetNowCurrent() | 同步 | int32_t (电流 mA) | `interfaces/inner_api/native/include/battery_srv_client.h:73` |
| GetRemainEnergy() | 同步 | int32_t (剩余电量 mAh) | `interfaces/inner_api/native/include/battery_srv_client.h:77` |
| GetTotalEnergy() | 同步 | int32_t (总电量 mAh) | `interfaces/inner_api/native/include/battery_srv_client.h:81` |
| GetCapacityLevel() | 同步 | BatteryCapacityLevel (电量等级) | `interfaces/inner_api/native/include/battery_srv_client.h:85` |
| GetRemainingChargeTime() | 同步 | int64_t (剩余充电时间秒) | `interfaces/inner_api/native/include/battery_srv_client.h:89` |

### 配置管理

**能力**: 提供场景化配置管理接口

| 接口 | 类型 | 说明 | 证据 |
|------|------|------|
| SetBatteryConfig() | 同步 | 设置电池配置（sceneName, value） | `interfaces/inner_api/native/include/battery_srv_client.h:93` |
| GetBatteryConfig() | 同步 | 获取电池配置（sceneName → result） | `interfaces/inner_api/native/include/battery_srv_client.h:97` |
| IsBatteryConfigSupported() | 同步 | 检查是否支持某配置 | `interfaces/inner_api/native/include/battery_srv_client.h:101` |

### 事件上报

**能力**: 通过 CommonEvent 向系统上报电池状态变化

| 事件名称 | 说明 | 证据 |
|---------|------|------|
| usual.event.BATTERY_CHANGED_INNER | 电池状态变化事件 | `interfaces/inner_api/native/include/battery_info.h:450` |

**事件字段**:
- soc - 电量百分比
- chargeState - 充电状态
- healthState - 健康状态
- pluggedType - 充电器类型
- voltage - 电压
- temperature - 温度
- technology - 技术类型
- present - 电池存在
- capacityLevel - 电量等级
- nowCurrent - 当前电流

**证据**: `interfaces/inner_api/native/include/battery_info.h:432-448`

### 统计事件上报

**能力**: 通过 HiSysEvent 上报电池统计信息

| 事件域 | 事件名称 | 说明 | 证据 |
|---------|---------|------|------|
| BATTERY | CHANGED | 电池信息变化统计 | `batterymgr.yaml:16-22` |
| BATTERY | ADJUST | 电池调节统计 | `batterymgr.yaml:23-28` |
| BATTERY | SLEEP_CURRENT | 休眠电流统计 | `batterymgr.yaml:29-31` |

---

## 依赖关系

### 上游依赖

| 依赖项 | 类型 | 说明 | 证据 |
|--------|------|------|------|
| drivers_interface_battery | HDI | 电池驱动接口，提供硬件数据 | `bundle.json:47` |
| common_event_service | 框架 | CommonEvent 服务，用于事件上报 | `bundle.json:43` |
| safwk | 框架 | System Ability 框架，SA 基础设施 | `bundle.json:67` |
| samgr | 框架 | SA 管理器，SA 注册和发现 | `bundle.json:68` |
| ipc | 框架 | IPC 通信基础设施 | `bundle.json:58` |

### 下游依赖

| 依赖项 | 类型 | 说明 |
|--------|------|------|
| powermgr_power_manager | 子系统 | 电源管理器，接收电池事件 |
| powermgr_battery_statistics | 子系统 | 电池统计，记录电池历史 |

### 内部依赖

| 模块 | 依赖模块 | 说明 | 证据 |
|------|--------|------|------|
| BatteryService | BatteryCallback, BatteryNotify | 服务依赖回调组件 | `services/native/include/battery_service.h:40-42` |
| BatteryService | HdiServiceStatusListener | 服务依赖 HDI 状态监听 | `services/native/include/battery_service.h:34` |
| BatteryNotify | distributed_notification_service | 通知依赖通知服务 | `services/BUILD.gn:84` |

---

## 适用场景

### 正常运行场景

1. **应用查询电池信息**
   - 应用通过 N-API `@ohos.batteryInfo` 查询电量、温度等
   - 应用通过 CommonEvent 订阅电池变化事件

2. **系统监控电池状态**
   - Battery Service 监听 HDI 电池回调
   - 状态变化时更新内部状态并触发 CommonEvent

3. **充电状态变化**
   - 插入充电器时，检测充电器类型和状态
   - 发送充电状态变化事件
   - 可选：播放充电提示音（需特性开关）

4. **低电量处理**
   - 电量低于警告阈值时，触发低电量通知
   - 可选：低于关机阈值时触发关机（需特性开关）

### 关机充电场景

1. **设备关机充电**
   - 启动充电进程（charger）
   - 显示充电动画和电量百分比
   - 检测电池状态变化更新界面

2. **低电量提示**
   - 显示"电池电量低"或"电池电量低，请连接电源"提示
   - 界面配置: `charger/sa_profile/animation.json:31-65`

---

## 技术约束

### 数据类型约束

| 数据类型 | 约束 | 证据 |
|---------|--------|------|
| BatteryChargeState | 枚举值范围 0-4 (CHARGE_STATE_BUTT) | `interfaces/inner_api/native/include/battery_info.h:33-58` |
| BatteryHealthState | 枚举值范围 0-6 (HEALTH_STATE_BUTT) | `interfaces/inner_api/native/include/battery_info.h:63-98` |
| BatteryPluggedType | 枚举值范围 0-4 (PLUGGED_TYPE_BUTT) | `interfaces/inner_api/native/include/battery_info.h:103-128` |
| BatteryCapacityLevel | 枚举值范围 0-8 (LEVEL_RESERVED) | `interfaces/inner_api/native/include/battery_info.h:133-178` |
| ChargeType | 枚举值范围 0-6 | `interfaces/inner_api/native/include/battery_info.h:183-213` |

### 错误码约束

| 错误码 | 值 | 说明 | 证据 |
|--------|-----|------|------|
| ERR_OK | 0 | 成功 | `interfaces/inner_api/native/include/battery_srv_errors.h` |
| ERR_FAILURE | 1 | 通用失败 | `interfaces/inner_api/native/include/battery_srv_errors.h` |
| ERR_PERMISSION_DENIED | 201 | 权限被拒绝 | `interfaces/inner_api/native/include/battery_srv_errors.h` |
| ERR_SYSTEM_API_DENIED | 202 | 系统 API 权限被拒绝 | `interfaces/inner_api/native/include/battery_srv_errors.h` |
| ERR_PARAM_INVALID | 401 | 无效输入参数 | `interfaces/inner_api/native/include/battery_srv_errors.h` |
| ERR_CONNECTION_FAIL | 5100101 | 连接服务失败 | `interfaces/inner_api/native/include/battery_srv_errors.h` |

---

## 相关跳转

- [系统架构](03_Architecture.md)
- [目录结构](02_Directory_Structure.md)
- [N-API 文档](04_NAPI_API.md)

---

**返回**: [导航](SUMMARY.md)
