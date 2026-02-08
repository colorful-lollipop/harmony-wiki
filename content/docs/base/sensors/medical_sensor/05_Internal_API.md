# 内部 API 文档

## 目的

本文档说明 Medical_Sensor 的内部模块接口、依赖方向和稳定性边界。

---

## 适用范围

本文档适用于需要：
- 理解模块间接口
- 进行模块重构或扩展
- 集成新传感器类型

---

## 核心模块接口

### 1. IPC 接口层（IMedicalSensorService）

**接口定义**：`frameworks/native/medical_sensor/include/i_medical_sensor_service.h`

**方法列表**：

| 方法签名 | 说明 | 参数 |
|---------|------|------|
| `EnableSensor(uint32_t, int64_t, int64_t)` | 启用传感器 | sensorId, samplingPeriodNs, maxReportDelayNs |
| `DisableSensor(uint32_t)` | 禁用传感器 | sensorId |
| `SetOption(uint32_t, uint32_t)` | 设置选项 | sensorId, opt |
| `GetSensorState(uint32_t)` | 获取状态 | sensorId |
| `RunCommand(uint32_t, uint32_t, uint32_t)` | 运行命令 | sensorId, cmdType, params |
| `GetSensorList()` | 获取传感器列表 | - |
| `TransferDataChannel(channel, client)` | 传输数据通道 | sensorBasicDataChannel, sensorClient |
| `DestroySensorChannel(client)` | 销毁通道 | sensorClient |

**证据**：
- 接口定义：`frameworks/native/medical_sensor/include/i_medical_sensor_service.h:30-64`

---

### 2. HDI 接口层（ISensorHdiConnection）

**接口定义**：`services/medical_sensor/hdi_connection/interface/include/sensor_hdi_connection.h`

**方法列表**：

| 方法签名 | 说明 | 参数 |
|---------|------|------|
| `ConnectHdi()` | 连接 HDI 服务 | - |
| `GetSensorList(vector)` | 获取传感器列表 | sensorList |
| `EnableSensor(int32_t)` | 启用传感器 | sensorId |
| `DisableSensor(int32_t)` | 禁用传感器 | sensorId |
| `SetBatch(int32_t, int64_t, int64_t)` | 设置批处理 | sensorId, samplingInterval, reportInterval |
| `SetMode(int32_t, int32_t)` | 设置模式 | sensorId, mode |
| `SetOption(int32_t, int32_t)` | 设置选项 | sensorId, option |
| `RunCommand(int32_t, int32_t, int32_t)` | 运行命令 | sensorId, cmd, params |
| `RegisteDataReport(cacheData, reportDataCache)` | 注册数据上报 | cacheData, reportDataCache |
| `DestroyHdiConnection()` | 销毁连接 | - |

**证据**：
- 接口定义：`services/medical_sensor/hdi_connection/interface/include/sensor_hdi_connection.h:26-58`

---

### 3. 权限检查接口（PermissionUtil）

**类定义**：`utils/include/permission_util.h`

**方法列表**：

| 方法签名 | 说明 | 参数 |
|---------|------|------|
| `CheckSensorPermission(token, sensorTypeId)` | 检查传感器权限 | callerToken, sensorTypeId |
| `AddPermissionRecord(token, permissionName, status)` | 记录权限使用 | tokenID, permissionName, status |

**证据**：
- 权限检查：`utils/src/permission_util.cpp:40-49`

---

## 数据结构定义

### SensorEvent

传感器数据事件结构，用于传递传感器数据。

**定义位置**：`interfaces/native/include/medical_native_type.h:97-105`

```cpp
typedef struct SensorEvent {
    int32_t sensorTypeId;   // 传感器类型 ID
    int32_t version;        // 传感器算法版本
    int64_t timestamp;      // 数据上报时间戳
    uint32_t option;        // 数据选项（测量范围和精度）
    int32_t mode;           // 数据上报模式
    uint8_t *data;          // 传感器数据
    uint32_t dataLen;       // 数据长度
} SensorEvent;
```

### MedicalSensor

传感器信息类，用于 IPC 传输。

**定义位置**：`utils/include/medical_sensor.h:26-68`

```cpp
class MedicalSensor : public Parcelable {
    uint32_t sensorId_;           // 传感器 ID
    std::string name_;            // 传感器名称
    std::string vendor_;          // 供应商
    uint32_t version_;            // 版本
    float maxRange_;              // 最大测量范围
    float resolution_;            // 分辨率
    uint32_t flags_;              // 标志
    int32_t fifoMaxEventCount_;   // FIFO 最大事件数
    int64_t minSamplePeriodNs_;   // 最小采样周期
    int64_t maxSamplePeriodNs_;   // 最大采样周期
};
```

### AppThreadInfo

应用线程信息，用于存储调用者身份。

**定义位置**：`utils/include/app_thread_info.h:24-31`

```cpp
struct AppThreadInfo {
    int32_t pid { 0 };
    int32_t uid { 0 };
    AccessTokenID callerToken { 0 };
};
```

---

## 模块依赖关系

### 依赖方向

```
interfaces/plugin/ (N-API)
    ↓
interfaces/native/ (Native API)
    ↓
frameworks/native/ (Client Framework)
    ↓ IPC
services/medical_sensor/ (Service)
    ↓ HDI
驱动层
```

### 循环依赖检查

**当前无循环依赖**：依赖方向为单向（上层依赖下层）。

---

## 稳定性边界

### 稳定接口（可安全依赖）

| 接口 | 说明 | 稳定性 |
|------|------|--------|
| `IMedicalSensorService` | IPC 接口定义 | ✅ 稳定（版本化） |
| `ISensorHdiConnection` | HDI 接口定义 | ✅ 稳定（系统接口） |
| `PermissionUtil` | 权限检查工具 | ✅ 稳定（公共工具） |
| Native API (`medical_native_impl.h`) | Native C API | ⚠️ 相对稳定（需向后兼容） |

### 不稳定接口（可能变更）

| 接口 | 说明 | 稳定性 |
|------|------|--------|
| `SensorHdiConnection` | HDI 适配层门面 | ⚠️ 可能随 HDI 版本变化 |
| `MedicalSensorService` | 系统能力实现 | ⚠️ 内部实现，可能重构 |

---

## 可替换点

### 1. HDI 适配层

**位置**：`services/medical_sensor/hdi_connection/`

**可替换性**：可以替换为不同的 HDI 实现（如兼容层、仿真层）。

**证据**：
- 适配器选择：`services/medical_sensor/hdi_connection/interface/src/sensor_hdi_connection.cpp:32-46`

### 2. 权限检查策略

**位置**：`utils/src/permission_util.cpp:35-38`

**可替换性**：可以通过修改 `sensorPermissions_` 映射表添加新的传感器类型和权限。

**证据**：
- 权限映射：`utils/src/permission_util.cpp:35-38`

### 3. 数据通道实现

**位置**：`frameworks/native/medical_sensor/include/medical_sensor_data_channel.h`

**可替换性**：可以替换为不同的数据通道实现（如共享内存、Socket、管道）。

---

## 关键调用链

### 订阅传感器数据调用链

```
应用层 (JS)
  └─> medical.on()
      └─> N-API 层 (On)
          └─> Native 层 (SubscribeSensor)
              └─> 创建数据通道 (CreateSensorDataChannel)
                  └─> IPC Proxy (MedicalSensorServiceClient)
                      └─> IPC 调用 (EnableSensor)
                          └─> 服务端 (MedicalSensorService)
                              └─> 权限检查 (CheckSensorPermission)
                                  └─> HDI 适配层 (EnableSensor)
                                      └─> 驱动层
```

### 传感器数据上报调用链

```
驱动层
  └─> HDI 事件回调 (OnDataEvent)
      └─> HDI 适配层
          └─> 服务端 (回调数据)
              └─> 数据通道 (TransferDataChannel)
                  └─> Native 层 (HandleSensorData)
                      └─> N-API 层 (DataCallbackImpl)
                          └─> 事件循环 (EmitUvEventLoop)
                              └─> JS 回调
```

---

## 相关跳转

- [项目概览](00_Overview.md) - 项目定位和核心能力
- [架构说明](02_Architecture.md) - 系统架构和数据流
- [N-API 参考](03_N-API_Reference.md) - JS API 文档
- [GN Targets](05_GN_Targets.md) - 构建目标和依赖图
