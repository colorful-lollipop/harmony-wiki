# Thermal Manager 项目概览

## 目的

本文档提供 thermal_manager 项目的总体概览，帮助新人快速理解项目定位、核心能力和关键概念。

## 适用范围

本文档适用于：
- OpenHarmony thermal_manager 模块
- 版本: main (2026-02-06)
- 目标平台: 标准系统 (standard system type)

## 相关文档

- [README.md](README.md) - 项目首页
- [01_Module_Boundaries.md](01_Module_Boundaries.md) - 模块边界与能力
- [02_Directory_Structure.md](02_Directory_Structure.md) - 目录结构说明
- [03_Architecture.md](03_Architecture.md) - 架构设计
- [04_NAPI_Interface.md](04_NAPI_Interface.md) - N-API 接口文档
- [08_Security_Review.md](08_Security_Review.md) - 安全风险评审

---

## 项目定位

### 核心职责

Thermal Manager 模块提供**设备温度管理和控制能力**，确保系统的热安全和用户体验。

主要功能：
1. **温度监控** - 实时监测设备各温度传感器数据
2. **热级别决策** - 基于温度传感器数据决策热级别（COOL/NORMAL/WARM/HOT 等）
3. **动作执行** - 根据热级别执行相应动作（降频、限流、关机等）
4. **回调通知** - 向应用和其他子系统提供温度变化通知

### 在系统中的位置

```
┌─────────────────────────────────────────────────────────────┐
│                  OpenHarmony 系统                     │
├─────────────────────────────────────────────────────────────┤
│                                                   │
│  ┌──────────────┐        ┌──────────────┐  │
│  │ N-API Layer │        │ Thermal Svc   │  │
│  │ (JS API)    │◄────►│ (SA 3303)   │  │
│  └──────────────┘        └──────────────┘  │
│         ▲                        │          │
│         │                        ▼          │
│  ┌───────────────────────────────────────┐   │
│  │ Applications / Other Subsystems      │   │
│  └───────────────────────────────────────┘   │
│                                       │
└───────────────────────────────────────────────┘
```

### 与其他子系统交互

从 `bundle.json` 的依赖关系，thermal_manager 与以下子系统交互：

| 子系统 | 交互内容 | 组件 |
|---|---|---|
| **display_manager** | 获取/控制屏幕状态 | brightness, display state |
| **battery_manager** | 获取电池状态、电量信息 | charging state, battery level |
| **soc_perf** | CPU/GPU 性能控制 | frequency limiting |
| **common_event_service** | 订阅/发布公共事件 | airplane mode, screen on/off |
| **power_manager** | 电源管理协作 | power policy |
| **window_manager** | 窗口管理（用于弹窗） | dialog popup |
| **audio_framework** | 音量控制（可选） | volume control |
| **netmanager_base** | 飞行模式管理（可选） | airplane mode |

---

## 核心能力

### 1. 温度查询

**能力描述**: 查询当前热级别

**证据**: `frameworks/napi/thermal_manager_napi.cpp:193`

```cpp
napi_value ThermalManagerNapi::GetThermalLevel(napi_env env, napi_callback_info info)
{
    ThermalLevel level = g_thermalMgrClient.GetThermalLevel();
    int32_t levelValue = static_cast<int32_t>(level);
    napi_value napiValue;
    NAPI_CALL(env, napi_create_int32(env, levelValue, &napiValue));
    return napiValue;
}
```

**热级别枚举**:
- `COOL` (0) - 正常/冷却
- `NORMAL` (1) - 正常
- `WARM` (2) - 温暖
- `HOT` (3) - 炎热
- `OVERHEATED` (4) - 过热
- `WARNING` (5) - 警告
- `EMERGENCY` (6) - 紧急
- `ESCAPE` (7) - 逃脱

### 2. 温度回调订阅

**能力描述**: 应用订阅热级别变化，接收回调通知

**证据**: `frameworks/napi/thermal_manager_napi.cpp:204`

```cpp
napi_value ThermalManagerNapi::SubscribeThermalLevel(napi_env env, napi_callback_info info)
{
    // 参数校验
    if (argc != MAX_ARGC || !NapiUtils::CheckValueType(env, argv[ARG_0], napi_function)) {
        return error.ThrowError(env, ThermalErrors::ERR_PARAM_INVALID);
    }
    // 更新回调引用
    g_thermalLevelCallback->UpdateCallback(env, argv[ARG_0]);
    // 订阅服务端回调
    g_thermalMgrClient.SubscribeThermalLevelCallback(g_thermalLevelCallback);
    return result;
}
```

**回调机制**: 使用 napi_send_event 异步通知 JS 回调

### 3. 温度传感器查询

**能力描述**: 查询特定传感器的温度值

**证据**: `interfaces/inner_api/native/include/thermal_mgr_client.h:40`

```cpp
int32_t GetThermalSensorTemp(const SensorType type);
```

**传感器类型**: SOC, BATTERY, SHELL, AMBIENT 等

### 4. 动作执行

**能力描述**: 根据热级别执行相应动作

**动作类型** (来源: `services/BUILD.gn`):
- CPU 降频 (big/medium/little core, boost, isolate)
- GPU 降频
- 电压/电流限制
- 屏幕亮度调节
- 音量降低
- 应用进程限制
- 飞行模式启用
- 关机
- 弹窗警告
- 热级别设置

---

## 运行环境

### 系统要求

- **OpenHarmony 版本**: API 9+
- **系统类型**: 标准系统 (standard)
- **处理器架构**: aarch64, x86_64

### 资源限制

从 `bundle.json`:
- **ROM**: 1024KB
- **RAM**: 2048KB

---

## 关键概念

### 1. System Ability (SA)

Thermal Manager 作为 System Ability 运行，SA ID 为 **3303**。

**证据**: `sa_profile/3303.json:5`

```json
{
    "name": 3303,
    "libpath": "libthermalservice.z.so",
    "run-on-create": true,
    "distributed": false
}
```

### 2. N-API (Node-API)

Node.js 应用接口层，提供 JS API 给应用使用。

**模块名**: "thermal"

**证据**: `frameworks/napi/thermal_manager_napi.cpp:286`

```cpp
static napi_module g_module = {
    .nm_version = 1,
    .nm_flags = 0,
    .nm_filename = "thermal",
    .nm_register_func = ThermalInit,
    .nm_modname = "thermal",
    .nm_priv = ((void*)0),
    .reserved = {0}
};
```

### 3. Thermal Policy

基于温度传感器数据决策热级别的策略引擎。

**证据**: `services/native/include/thermal_policy/thermal_policy.h:40`

**策略流程**:
1. 传感器温度上报
2. 传感器集群（Sensor Cluster）级别决策
3. 策略匹配（Policy + State）
4. 动作执行

### 4. Thermal Observer

观察者模式实现，订阅温度变化并触发策略执行。

**证据**: `services/native/include/thermal_observer/thermal_observer.h:34`

### 5. Thermal Protector

非运行态（关机/重启）下的简化温度控制。

**证据**: `application/protector/` 目录

### 6. HDI (Hardware Driver Interface)

与底层热驱动通信的接口。

**服务名**: thermal_interface_service

**证据**: `services/native/src/thermal_service.cpp:57`

---

## 系统能力

- **系统能力 ID**: `SystemCapability.PowerManager.ThermalManager`
- **子系统**: powermgr
- **组件名**: thermal_manager

---

## 配置系统

### 配置文件路径

1. `/vendor/etc/thermal_config/thermal_service_config.xml` - 厂商配置（优先）
2. `/system/etc/thermal_config/thermal_service_config.xml` - 系统配置
3. `etc/thermal_config/thermal_service_config.xml` - 默认配置

**证据**: `services/native/src/thermal_service.cpp:54-56`

### 配置结构

```
<thermal version="0.99" product="ipx">
    <base>        <!-- 基础参数 -->
    <level>       <!-- 温度级别定义 -->
    <state>       <!-- 状态机定义 -->
    <action>      <!-- 动作定义 -->
    <policy>      <!-- 策略规则 -->
</thermal>
```

详细说明参见 README.md。

---

## 架构层级

```
┌─────────────────────────────────────────────────────┐
│              Applications (JS/TS/ArkTS)       │
├─────────────────────────────────────────────────────┤
│                                                   │
│  ┌───────────────────────────────────────────┐   │
│  │         N-API Layer (thermal)        │   │
│  │  - thermal_manager_napi.cpp          │   │
│  │  - ThermalLevelCallback              │   │
│  └───────────────────────────────────────────┘   │
│                    ▲ IPC (Binder)            │
│                    │                         │
│  ┌───────────────────────────────────────┐   │
│  │   Thermal Service (SA 3303)       │   │
│  │  ┌───────────┐  ┌──────────┐  │   │
│  │  │ Observer  │  │  Policy  │  │   │
│  │  └───────────┘  └──────────┘  │   │
│  │       ▲              ▲            │   │
│  │       │              │            │   │
│  │  ┌──────────────┐             │   │
│  │  │ Action Mgr   │             │   │
│  │  │ - CPU, GPU,  │             │   │
│  │  │   Voltage,    │             │   │
│  │  │   Current,    │             │   │
│  │  │   ...         │             │   │
│  │  └──────────────┘             │   │
│  └───────────────────────────────────────┘   │
│                    ▲ HDI                  │
│                    │                         │
│  ┌───────────────────────────────────┐   │
│  │   Thermal Drivers (HDF)          │   │
│  │   - Temperature Sensors            │   │
│  │   - Fan Control                  │   │
│  └───────────────────────────────────┘   │
└─────────────────────────────────────────────┘
```

---

## 总结

Thermal Manager 是 OpenHarmony 电源管理子系统的核心组件，通过 N-API 提供温度查询和回调功能，作为 System Ability (SA 3303) 运行，与显示、电池、SOC 性能等多个子系统协同工作，确保设备热安全和良好用户体验。

**关键特点**:
- ✅ 标准化 N-API 接口
- ✅ 配置驱动的策略引擎
- ✅ 观察者模式的事件驱动架构
- ✅ 多动作支持（CPU/GPU 降频、电压/电流限制、关机等）
- ✅ 非运行态保护器
- ✅ HDI 驱动接口标准化
