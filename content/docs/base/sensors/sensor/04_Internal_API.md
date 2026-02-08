# 内部模块接口

> **目的**: 描述 Sensor 子系统的内部模块接口、依赖方向、稳定性和可替换点
> **适用范围**: /base/sensors/sensor（排除 test/ 目录）
> **关键结论**: Sensor 子系统内部接口分为 Framework 层 API 和 Service 层 API，通过 IDL 定义的 IPC 接口进行通信
> **相关跳转**: [目录结构](01_Directory_Structure.md) | [架构说明](02_Architecture.md)

---

## Framework 层接口

### SensorAgent 接口

**位置**: `interfaces/inner_api/sensor_agent.h`

**稳定性**: **稳定接口** - 系统内部 API

**主要方法**:

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `SubscribeSensor()` | sensorTypeId, callback | int32_t | 订阅传感器 |
| `UnsubscribeSensor()` | sensorTypeId | int32_t | 取消订阅 |

**依赖**: 无外部依赖（基础接口）

**调用方向**:
```
上层 (N-API/CJ/Taihe) → SensorAgent → SensorServiceClient → IPC
```

> **证据**: `interfaces/inner_api/sensor_agent.h`

---

## Service 层接口

### ISensorService 接口 (IDL)

**位置**: `frameworks/native/ISensorService.idl`

**稳定性**: **稳定接口** - IDL 定义的 IPC 接口

**主要方法**:

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `EnableSensor()` | SensorDescriptionIPC, samplingPeriodNs, maxReportDelayNs | void | 启用传感器 |
| `DisableSensor()` | SensorDescriptionIPC | void | 禁用传感器 |
| `GetSensorList()` | - | Sensor[] | 获取传感器列表 |
| `GetSensorListByDevice()` | deviceId | Sensor[] | 获取设备传感器列表 |
| `TransferDataChannel()` | FileDescriptor, sensorClient | void | 传输数据通道 |
| `DestroySensorChannel()` | sensorClient | void | 销毁数据通道 |
| `SuspendSensors()` | pid | void | 暂停传感器 |
| `ResumeSensors()` | pid | void | 恢复传感器 |
| `GetActiveInfoList()` | pid | ActiveInfo[] | 获取活跃信息列表 |
| `CreateSocketChannel()` | sensorClient | void | 创建 Socket 通道 |
| `DestroySocketChannel()` | sensorClient | void | 销毁 Socket 通道 |
| `EnableActiveInfoCB()` | - | void | 启用活跃信息回调 |
| `DisableActiveInfoCB()` | - | void | 禁用活跃信息回调 |
| `ResetSensors()` | - | void | 重置传感器 |
| `SetDeviceStatus()` | deviceStatus | void | 设置设备状态 |
| `TransferClientRemoteObject()` | sensorClient | void | 传输客户端远程对象 |
| `DestroyClientRemoteObject()` | sensorClient | void | 销毁客户端远程对象 |

**依赖**: 无外部依赖（IDL 生成的接口）

**调用方向**:
```
Client (libsensor_client) → IPC → Service (libsensor_service)
```

> **证据**: `frameworks/native/ISensorService.idl`

### ISensorClient 接口 (回调)

**位置**: `frameworks/native/include/i_sensor_client.h`

**稳定性**: **稳定接口** - Service 到 Client 的回调接口

**主要方法**:

| 方法名 | 参数 | 返回值 | 说明 |
|--------|------|--------|------|
| `ProcessPlugEvent()` | SensorPlugData | int32_t | 处理传感器插拔事件 |

**依赖**: 无外部依赖

**调用方向**:
```
Service → IPC → Client (SensorClientStub) → N-API/JS
```

> **证据**: `frameworks/native/include/i_sensor_client.h`

---

## HDI 接口

### ISensorHdiConnection 接口

**位置**: `services/hdi_connection/interface/include/i_sensor_hdi_connection.h`

**稳定性**: **内部接口** - 连接到 HDF 驱动的接口

**主要方法** (待补充):

| 方法名 | 说明 |
|--------|------|
| `Connect()` | 连接到 HDI 服务 |
| `EnableSensor()` | 启用传感器 |
| `DisableSensor()` | 禁用传感器 |
| `SetBatch()` | 设置批量模式 |
| `SetMode()` | 设置传感器模式 |

**依赖**: HDF 驱动接口

**调用方向**:
```
SensorService → HDI Connection → HDF Driver
```

> **证据**: `services/hdi_connection/interface/include/i_sensor_hdi_connection.h`

---

## 数据通道接口

### SensorBasicDataChannel 接口

**位置**: `utils/common/include/sensor_basic_data_channel.h`

**稳定性**: **内部接口** - 数据通道基类

**主要方法** (待补充):

| 方法名 | 说明 |
|--------|------|
| `GetSensorDataChannel()` | 获取传感器数据通道 |
| `SetAccessTokenId()` | 设置访问令牌 ID |
| `GetAccessTokenId()` | 获取访问令牌 ID |

**依赖**: 无外部依赖

**调用方向**:
```
上层 (SensorAgent/SensorServiceClient) → SensorBasicDataChannel → StreamSocket/StreamSession
```

> **证据**: `utils/common/include/sensor_basic_data_channel.h`

---

## 接口依赖方向

### 自顶向下依赖

```
应用层
    ↓
N-API / Native API (JS/CJ/Taihe)
    ↓
SensorAgent (interfaces/inner_api)
    ↓
SensorServiceClient (IPC 客户端)
    ↓
IPC (Binder)
    ↓
SensorService (System Ability 3601)
    ↓
SensorManager / SensorDataManager
    ↓
HDI Connection
    ↓
HDF Driver (硬件驱动层)
```

### 依赖原则

- **单向依赖**: 下层不依赖上层
- **无循环依赖**: 避免模块间循环依赖
- **清晰边界**: 每层有明确的职责边界

> **证据**: 各 BUILD.gn 文件中的 `deps` 配置

---

## 接口稳定性

### 稳定接口 (Stable Interface)

**定义**: API 稳定性等级，标识哪些接口是稳定的，不会频繁变更

| 接口 | 稳定性 | 证据 | 说明 |
|------|---------|------|------|
| `SensorAgent` | 稳定 | `interfaces/inner_api/sensor_agent.h` | 系统内部 API，向后兼容 |
| `ISensorService` | 稳定 | `frameworks/native/ISensorService.idl` | IDL 定义的 IPC 接口 |
| `ISensorClient` | 稳定 | `frameworks/native/include/i_sensor_client.h` | IPC 回调接口 |

### 不稳定接口 (Unstable Interface)

**定义**: 内部实现细节，可能随时变更

| 接口 | 稳定性 | 证据 | 说明 |
|------|---------|------|------|
| `SensorManager` 内部方法 | 不稳定 | `services/src/sensor_manager.cpp` | 实现细节，可能重构 |
| `SensorDataManager` 内部方法 | 不稳定 | `services/src/sensor_data_manager.cpp` | 实现细节，可能重构 |
| `HDI Connection` 内部方法 | 不稳定 | `services/hdi_connection/` | 驱动相关，可能变更 |

**稳定性判断依据**:
- **位置**: `interfaces/` 目录下的头文件是稳定接口
- **命名**: 带 `I` 前缀的接口类是稳定的
- **注释**: 有 `@SystemApi` 或类似稳定注解的接口
- **依赖关系**: 跨层接口通常更稳定

> **证据**: 目录结构、命名约定、include 层级

---

## 可替换点

### 可替换的组件

以下组件设计为可替换，方便测试和定制：

| 组件 | 替换方式 | 难度 |
|------|----------|--------|
| HDI Connection | 实现不同的 HDI 适配器 | 中 - 需要匹配接口 |
| 数据传输通道 | 替换 StreamSocket/StreamSession | 中 - 需要匹配接口 |
| 权限验证模块 | 替换 PermissionUtil | 低 - 接口清晰 |
| 日志模块 | 替换 SensorLog | 低 - 接口清晰 |

### 不可替换的组件

以下组件紧密耦合，难以替换：

| 组件 | 替换难度 | 原因 |
|------|----------|------|
| SensorService (SA) | 高 - System Ability 注册和 IPC 集成 |
| IPC 机制 (Binder) | 高 - 涉及系统底层通信 |
| IDL 接口 | 中 - 需要重新生成 Stub/Proxy 代码 |

> **TODO**: 补充更多可替换点分析

---

## 版本兼容性

### API 版本控制

| API 层 | 版本控制方式 |
|---------|------------|
| N-API (JS) | 通过 @ohos.sensor 模块版本控制 |
| Native API | 通过 NDK 版本控制 (libsensor.json, min_compact_version=6) |
| IDL 接口 | 通过版本号控制 (libsensor_proxy_3.0) |
| HDI 接口 | 通过版本号控制 |

> **证据**: `frameworks/native/BUILD.gn:104-108`, `services/BUILD.gn:133`

### ABI 兼容性

- **C++ ABI**: libsensor_client.z.so, libsensor_service.z.so
- **N-API ABI**: libsensor.z.so
- **NDK ABI**: sensor.so, ohsensor.so
- **CJ FFI ABI**: cj_sensor_ffi.z.so
- **Taihe ETS ABI**: sensor_taihe_native.z.so, sensor_abc.abc

> **证据**: 各 BUILD.gn 文件中的 target 类型

---

## 内部枚举和常量

### 传感器类型枚举

**位置**: `interfaces/inner_api/sensor_agent_type.h`

**稳定性**: 稳定 - 传感器类型定义不会频繁变更

**主要类型**:
- `SENSOR_TYPE_ID_*` - 传感器类型 ID
- `SensorInfo` - 传感器信息结构
- `SensorData` - 传感器数据结构

> **证据**: `interfaces/inner_api/sensor_agent_type.h`

### 错误码枚举

**位置**: `utils/common/include/sensor_errors.h`

**稳定性**: 稳定 - 错误码定义

**主要错误**:
- `ERR_OK` - 成功
- `PERMISSION_DENIED` - 权限拒绝
- `PARAMETER_ERROR` - 参数错误
- `SENSOR_NATIVE_GET_SERVICE_ERR` - 获取服务失败

> **证据**: `utils/common/include/sensor_errors.h`

---

## 相关跳转

- [架构说明](02_Architecture.md) - 查看组件交互和数据流
- [对外 N-API 参考](03_NAPI_Reference.md) - 查看对外 API
- [目录结构](01_Directory_Structure.md) - 查看模块组织
