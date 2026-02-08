# OpenHarmony Sensor 子系统概览

> **目的**: 描述 Sensor 子系统的项目定位、边界、核心能力和运行环境
> **适用范围**: OpenHarmony 标准系统的 Sensor 子系统（base/sensors/sensor）
> **关键结论**: Sensor 子系统是一个统一的传感器管理框架，为上层应用提供低时延、低功耗的感知数据服务
> **相关跳转**: [目录结构](01_Directory_Structure.md) | [N-API 参考](03_NAPI_Reference.md) | [架构说明](02_Architecture.md)

---

## 项目定位

### 设计目标

Sensor 子系统是 OpenHarmony "泛 Sensor 服务子系统" 的核心组件，旨在：

1. **统一管理**: 提供统一的传感器管理框架，支持多种传感器类型
2. **低时延**: 通过优化的数据通道实现低时延数据上报
3. **低功耗**: 智能的电源策略管理，减少不必要的传感器唤醒
4. **跨进程**: 支持多进程访问传感器数据，通过 System Ability 实现 IPC 通信
5. **多语言支持**: 提供 JavaScript (N-API)、C++ (Native)、ArkTS (ETS/Taihe)、Cangjie (CJ/FFI) 等多语言 API

### 适用场景

- **1+8+N 产品**: 支撑 OpenHarmony "1+8+N" 全场景智慧化战略的传感器需求
- **IoT 设备**: 为物联网设备提供传感器数据感知能力
- **可穿戴设备**: 为手表、手环等可穿戴设备提供运动、健康等传感器
- **智能家居**: 为智能音箱、智能家电等提供环境、接近光等传感器

---

## 项目边界

### 包含范围

**本项目包含**:
- 传感器客户端框架
- 传感器服务实现
- JS/TS/CJ 等多语言绑定
- 权限管理和验证
- 数据通道管理
- 音频转震动模块

**不包含**:
- 传感器硬件驱动层（位于 HDF drivers 接口）
- 传感器 HAL 实现（位于 driver framework）
- 杂项设备子系统（sensors_miscdevice）
- 测试代码（test/ 目录）

### 依赖子系统

从 `bundle.json` 中提取的依赖组件：

| 子系统/组件 | 用途 | 依赖类型 |
|------------|------|---------|
| `bundle_framework` | 应用框架 | 构建依赖 |
| `common_event_service` | 公共事件服务 | 运行时依赖 |
| `c_utils` | C 工具库 | 构建依赖 |
| `data_share` | 数据共享 | 运行时依赖 |
| `hilog` | 日志服务 | 运行时依赖 |
| `build_framework` | 构建框架 | 构建依赖 |
| `hisysevent` | 系统事件 | 运行时依赖 |
| `napi` | N-API 绑定 | 构建依赖 |
| `drivers_interface_sensor` | 传感器驱动接口 | 运行时依赖 |
| `access_token` | 访问令牌 | 运时依赖 |
| `hitrace` | 性能追踪 | 运行时依赖 |
| `ipc` | IPC 框架 | 运行时依赖 |
| `memmgr` | 内存管理 | 运行时依赖 |
| `safwk` | 系统能力框架 | 运行时依赖 |
| `samgr` | 系统能力管理器 | 运行时依赖 |
| `eventhandler` | 事件处理器 | 运行时依赖 |
| `hicollie` | 崩溃监控 | 运行时依赖 |
| `init` | Init 进程 | 运行时依赖 |
| `selinux_adapter` | SELinux 适配 | 运行时依赖 |
| `cJSON` | JSON 解析 | 构建依赖 |
| `runtime_core` | 运行时核心 | 运行时依赖 |
| `os_account` | 系统账户 | 运行时依赖 |

---

## 核心能力

### 支持的传感器类型

基于 `frameworks/js/napi/src/sensor_js.cpp` 和 `interfaces/inner_api/sensor_agent_type.h`，Sensor 子系统支持以下传感器类型：

#### 运动类

| 传感器类型 | SensorTypeId | 说明 | 权限要求 |
|----------|------------|------|-----------|
| 加速度传感器 | SENSOR_TYPE_ID_ACCELEROMETER | ohos.permission.ACCELEROMETER |
| 加速度未校准 | SENSOR_TYPE_ID_ACCELEROMETER_UNCALIBRATED | ohos.permission.ACCELEROMETER |
| 线性加速度 | SENSOR_TYPE_ID_LINEAR_ACCELERATION | ohos.permission.ACCELEROMETER |
| 陀螺仪 | SENSOR_TYPE_ID_GYROSCOPE | ohos.permission.GYROSCOPE |
| 陀螺仪未校准 | SENSOR_TYPE_ID_GYROSCOPE_UNCALIBRATED | ohos.permission.GYROSCOPE |
| 重力传感器 | SENSOR_TYPE_ID_GRAVITY | 无 |
| 线性加速度计 | SENSOR_TYPE_ID_LINEAR_ACCELEROMETER | 无 |

#### 姿态类

| 传感器类型 | SensorTypeId | 说明 | 权限要求 |
|----------|------------|------|-----------|
| 旋转矢量 | SENSOR_TYPE_ID_ROTATION_VECTOR | 无 |
| 方向传感器 | SENSOR_TYPE_ID_ORIENTATION | 无 |
| 游戏旋转矢量 | SENSOR_TYPE_ID_GAME_ROTATION_VECTOR | 无 |

#### 环境类

| 传感器类型 | SensorTypeId | 说明 | 权限要求 |
|----------|------------|------|-----------|
| 磁力计 | SENSOR_TYPE_ID_MAGNETIC_FIELD | 无 |
| 气压传感器 | SENSOR_TYPE_ID_BAROMETER | 无 |
| 湿度传感器 | SENSOR_TYPE_ID_HUMIDITY | 无 |

#### 光线类

| 传感器类型 | SensorTypeId | 说明 | 权限要求 |
|----------|------------|------|-----------|
| 环境光传感器 | SENSOR_TYPE_ID_AMBIENT_LIGHT | 无 |
| 接近光传感器 | SENSOR_TYPE_ID_PROXIMITY | 无 |
| 色温传感器 | SENSOR_TYPE_ID_COLOR_TEMPERATURE | 无 |

#### 健康类

| 传感器类型 | SensorTypeId | 说明 | 权限要求 |
|----------|------------|------|-----------|
| 心率传感器 | SENSOR_TYPE_ID_HEART_RATE | ohos.permission.READ_HEALTH_DATA |

#### 其他类

| 传感器类型 | SensorTypeId | 说明 | 权限要求 |
|----------|------------|------|-----------|
| 霍尔传感器 | SENSOR_TYPE_ID_HALL | 无 |
| 手握传感器 | SENSOR_TYPE_ID_SIGNIFICANT_MOTION | ohos.permission.ACTIVITY_MOTION |
| 计步器 | SENSOR_TYPE_ID_PEDOMETER | ohos.permission.ACTIVITY_MOTION |
| 身体状态传感器 | SENSOR_TYPE_ID_BODY_STATE | ohos.permission.ACTIVITY_MOTION |
| 设备方向传感器 | SENSOR_TYPE_ID_DEVICE_ORIENTATION | 无 |

> **证据来源**: `frameworks/js/napi/src/sensor_js.cpp` (CreateEnumSensorId 函数), `interfaces/inner_api/sensor_agent_type.h`

---

## 运行环境

### System Ability 配置

**SA ID**: 3601 (SENSOR_SERVICE_ABILITY_ID)
**进程名**: "sensors"
**动态库**: libsensor_service.z.so
**配置文件**: `sa_profile/3601.json`

> **证据**: `sa_profile/3601.json`, `services/src/sensor_service.cpp:51`

### 系统能力 (SysCap)

从 `bundle.json` 中提取：
- `SystemCapability.Sensors.Sensor` - 标准传感器能力
- `SystemCapability.Sensors.Sensor.Lite` - 轻量级传感器能力

> **证据**: `bundle.json:12`

### 构建配置

**ROM 占用**: 2048KB
**RAM 占用**: ~4096KB
**适配系统类型**: standard

> **证据**: `bundle.json:15-16`

---

## 关键概念

### 传感器订阅模型

Sensor 子系统采用事件订阅模式：

1. **订阅 (subscribe)**:
   - `on(type, callback, options)` - 持续监听传感器数据变化
   - `once(type, callback)` - 单次监听传感器数据

2. **取消订阅 (unsubscribe)**:
   - `off(type, callback)` - 停止监听传感器数据

3. **数据上报**:
   - 通过 IPC 回调将传感器数据从服务传递到客户端
   - 支持自定义采样间隔 (interval)

### 权限模型

**权限类型**:

| 权限类型 | 敏感度 | 说明 | 传感器类型 |
|----------|--------|------|-----------|
| system_grant | 系统授权 | ACCELEROMETER, GYROSCOPE |
| user_grant | 用户授权 | ACTIVITY_MOTION, READ_HEALTH_DATA |

**权限检查点**:
1. **客户端订阅时**: `PermissionUtil::CheckSensorPermission()`
2. **服务端启用传感器时**: `SensorService::CheckAuthAndParameter()`
3. **动态权限变更**: 注册权限状态回调，处理运行时权限撤销

> **证据**: `utils/common/src/permission_util.cpp`, `services/src/sensor_service.cpp`

### 数据通道

Sensor 支持两种数据传输方式：

1. **FileDescriptor 通道**: 高性能数据流，用于传感器数据上报
2. **Socket 通道**: 用于远程设备传感器数据传输

> **证据**: `interfaces/inner_api/sensor_basic_data_channel.h`, `utils/ipc/src/stream_socket.cpp`

---

## 架构概览

Sensor 子系统采用分层架构：

```
应用层 (JS/TS/CJ 应用)
    ↓
N-API 绑定层 (frameworks/js/napi, frameworks/cj, frameworks/ets/taihe)
    ↓
Native 客户端 (frameworks/native)
    ↓
IPC 层 (HDF drivers_interface_sensor)
    ↓
Sensor 服务 (services)
    ↓
HDI 层 (硬件驱动接口)
```

详细架构说明请参考：[架构说明](02_Architecture.md)

---

## 相关跳转

- [目录结构与模块职责](01_Directory_Structure.md) - 了解代码组织
- [对外 N-API 完整参考](03_NAPI_Reference.md) - 学习 JS API 使用
- [内部模块接口](04_Internal_API.md) - 查看内部 API 定义
- [GN 目标梳理](05_GN_Targets.md) - 了解构建系统
- [安全风险评审](07_Security_Audit.md) - 查看安全分析

---

## 参考资料

- **主 README**: `/base/sensors/sensor/README.md` (英文)
- **中文 README**: `/base/sensors/sensor/README_zh.md` (中文)
- ** bundle.json**: `/base/sensors/sensor/bundle.json` - 组件配置
- **sensor.gni**: `/base/sensors/sensor/sensor.gni` - 构建配置
