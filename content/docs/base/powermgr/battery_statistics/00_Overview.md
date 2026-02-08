# 项目概览

## 项目定位

`battery_statistics` 是 OpenHarmony **电源管理子系统**（powermgr）的核心模块之一，负责收集和统计系统的**耗电信息**。

**证据来源**：`bundle.json:4` - `"description": "耗电统计服务，包括硬件耗电统计和软件耗电统计。"`

## 核心能力

### 1. 软件耗电统计

按 **UID**（用户 ID/应用 ID）统计各应用的软件耗电量。

**统计项**：
- CPU 耗电
- 运行锁（WakeLock）耗电
- 移动无线（Radio）耗电
- Wi-Fi 耗电
- GNSS（定位）耗电
- 传感器耗电
- 相机耗电
- 手电筒耗电
- 音频耗电
- 闹钟耗电

**证据来源**：`README.md:9-13`

### 2. 硬件耗电统计

除软件耗电外的所有硬件耗电。

**统计项**：
- 用户活动耗电（USER）
- 通话耗电（PHONE）
- 屏幕耗电（SCREEN）
- 蓝牙耗电（BLUETOOTH）
- Wi-Fi 耗电（WIFI）

**证据来源**：`README.md:14-18`

## 运行环境

| 属性 | 值 |
|------|-----|
| **运行平台** | OpenHarmony 标准系统（standard） |
| **系统能力** | SystemCapability.PowerManager.BatteryStatistics |
| **依赖子系统** | powermgr, ability, ipc, samgr |
| **运行进程** | powermgr（与 power_manager 同进程） |

**证据来源**：`bundle.json:17-21`

## 关键概念

### ConsumptionType（耗电类型）

枚举定义了 9 种主要耗电类型，用于区分不同的耗电来源。

| 枚举值 | 整数值 | 含义 |
|--------|--------|------|
| CONSUMPTION_TYPE_INVALID | 0 | 无效类型 |
| CONSUMPTION_TYPE_APP | 1 | 应用软件耗电 |
| CONSUMPTION_TYPE_BLUETOOTH | 2 | 蓝牙耗电 |
| CONSUMPTION_TYPE_IDLE | 3 | 空闲状态耗电 |
| CONSUMPTION_TYPE_PHONE | 4 | 通话耗电 |
| CONSUMPTION_TYPE_RADIO | 5 | 移动无线耗电 |
| CONSUMPTION_TYPE_SCREEN | 6 | 屏幕耗电 |
| CONSUMPTION_TYPE_USER | 7 | 用户活动耗电 |
| CONSUMPTION_TYPE_WIFI | 8 | Wi-Fi 耗电 |

**证据来源**：`frameworks/napi/src/battery_stats_module.cpp:113-123`

### 实体类（Entity）

模块采用**实体类模式**（Entity Pattern）跟踪各组件的耗电情况。每个实体负责特定硬件/软件组件的耗电计算。

**15 个实体类**：
- 基础实体：`BatteryStatsEntity`
- CPU 类：`CpuEntity`
- 网络类：`WifiEntity`, `BluetoothEntity`, `PhoneEntity`
- 定位类：`GnssEntity`
- 传感器类：`SensorEntity`, `IdleEntity`
- 显示类：`ScreenEntity`
- 多媒体类：`AudioEntity`, `CameraEntity`, `FlashlightEntity`
- 系统类：`WakelockEntity`, `AlarmEntity`
- 用户类：`UidEntity`, `UserEntity`

**证据来源**：`services/native/include/entities/*.h`

### System Ability

作为 **System Ability（SA）** 运行，提供跨进程服务能力。

| 属性 | 值 |
|------|-----|
| **SA ID** | 3304 |
| **进程** | powermgr |
| **库路径** | libbatterystats_service.z.so |
| **分布式** | false |
| **按需创建** | false |

**证据来源**：`sa_profile/3304.json`

## 系统集成

### 依赖关系

```
battery_statistics
├── battery_manager     # 电池状态查询
├── power_manager       # 电源管理
├── ability_runtime     # 能力运行时
├── ipc                # 进程间通信
├── samgr              # SA 管理
├── hilog              # 日志
├── hisysevent          # 系统事件
└── hicollie           # 看门狗
```

**证据来源**：`bundle.json:27-51`

### 相关仓库

| 仓库 | 说明 |
|------|------|
| powermgr_power_manager | 电源管理 |
| powermgr_battery_manager | 电池管理 |
| powermgr_display_manager | 显示管理 |
| powermgr_thermal_manager | 热管理 |

**证据来源**：`README.md:42-55`

## 版本历史

| 版本 | 说明 |
|------|------|
| 3.1 | 当前版本 |
| - | 支持标准系统 |
| - | 支持 N-API + ArkTS 双接口 |

## 资源占用

| 资源 | 限制 |
|------|------|
| ROM | 1024 KB |
| RAM | 2048 KB |

**证据来源**：`bundle.json:22-23`

## 相关文档

- [目录结构](01_Directory_Structure.md) - 详细文件组织
- [架构设计](02_Architecture.md) - 组件与数据流
- [N-API 接口](03_NAPI.md) - JS API 清单
- [安全评审](07_Security.md) - 风险分析
