# Native C API 参考

## API 概述

sensor_lite 提供 **8 个 Native C API**，全部定义在 `sensor_agent.h` 中。

### API 列表

| # | 函数 | 功能 | 同步/异步 |
|---|------|------|----------|
| 1 | `GetAllSensors` | 获取所有传感器信息 | 同步 |
| 2 | `ActivateSensor` | 使能传感器 | 同步 |
| 3 | `DeactivateSensor` | 去使能传感器 | 同步 |
| 4 | `SetBatch` | 设置采样/上报间隔 | 同步 |
| 5 | `SubscribeSensor` | 订阅传感器数据 | 同步 |
| 6 | `UnsubscribeSensor` | 取消订阅 | 同步 |
| 7 | `SetMode` | 设置数据上报模式 | 同步 |
| 8 | `SetOption` | 设置传感器选项 | 同步 |

### 头文件

```c
#include "sensor_agent.h"
#include "sensor_agent_type.h"
```

---

## API 详细说明

### GetAllSensors

```c
int32_t GetAllSensors(SensorInfo **sensorInfo, int32_t *count);
```

**功能**: 获取系统中所有传感器的信息

**参数**:
- `sensorInfo` - 输出参数，指向传感器信息数组的指针的指针
- `count` - 输出参数，指向传感器数量的指针

**返回值**:
- `SENSOR_OK` (0) - 成功
- `SENSOR_ERROR_INVALID_PARAM` - 参数无效
- 其他错误码 - 详见错误码表

**使用示例**:
```c
SensorInfo *sensorInfo = NULL;
int32_t count = 0;
int32_t ret = GetAllSensors(&sensorInfo, &count);
if (ret == SENSOR_OK) {
    for (int32_t i = 0; i < count; i++) {
        printf("Sensor: %s, Vendor: %s\n", 
               sensorInfo[i].sensorName,
               sensorInfo[i].vendorName);
    }
}
```

**IPC funcId**: 0

**证据**: `interfaces/kits/native/include/sensor_agent.h:59`

---

### ActivateSensor

```c
int32_t ActivateSensor(int32_t sensorTypeId, SensorUser *user);
```

**功能**: 使能指定的传感器。只有传感器使能后，订阅该传感器的用户才能获取数据。

**参数**:
- `sensorTypeId` - 传感器类型 ID
- `user` - 传感器用户信息

**返回值**:
- `SENSOR_OK` (0) - 成功
- `SENSOR_ERROR_INVALID_ID` - 无效的传感器 ID
- `SENSOR_ERROR_INVALID_PARAM` - 无效参数

**IPC funcId**: 1

**证据**: `interfaces/kits/native/include/sensor_agent.h:110`

---

### DeactivateSensor

```c
int32_t DeactivateSensor(int32_t sensorTypeId, SensorUser *user);
```

**功能**: 去使能指定的传感器

**参数**:
- `sensorTypeId` - 传感器类型 ID
- `user` - 传感器用户信息

**返回值**:
- `SENSOR_OK` (0) - 成功
- `SENSOR_ERROR_INVALID_ID` - 无效的传感器 ID
- `SENSOR_ERROR_INVALID_PARAM` - 无效参数

**IPC funcId**: 2

**证据**: `interfaces/kits/native/include/sensor_agent.h:122`

---

### SetBatch

```c
int32_t SetBatch(int32_t sensorTypeId, SensorUser *user, 
                 int64_t samplingInterval, int64_t reportInterval);
```

**功能**: 设置传感器的数据采样间隔和数据上报间隔

**参数**:
- `sensorTypeId` - 传感器类型 ID
- `user` - 传感器用户信息
- `samplingInterval` - 采样间隔（纳秒）
- `reportInterval` - 上报间隔（纳秒）

**返回值**:
- `SENSOR_OK` (0) - 成功
- `SENSOR_ERROR_INVALID_ID` - 无效的传感器 ID
- `SENSOR_ERROR_INVALID_PARAM` - 无效参数（间隔为负数）

**IPC funcId**: 3

**证据**: `interfaces/kits/native/include/sensor_agent.h:97`

**注意**: 当前实现中，仅在 `sensor_service_impl.c` 中进行了参数校验，未实际调用底层 HDI 设置间隔。

---

### SubscribeSensor

```c
int32_t SubscribeSensor(int32_t sensorTypeId, SensorUser *user);
```

**功能**: 订阅传感器数据。系统会将获取到的传感器数据上报给订阅者。

**参数**:
- `sensorTypeId` - 传感器类型 ID
- `user` - 传感器用户信息，包含回调函数

**返回值**:
- `SENSOR_OK` (0) - 成功
- `SENSOR_ERROR_INVALID_ID` - 无效的传感器 ID
- `SENSOR_ERROR_INVALID_PARAM` - 无效参数

**IPC funcId**: 4

**使用示例**:
```c
void SensorDataCallback(SensorEvent *event)
{
    float *data = (float *)event->data;
    // 处理传感器数据
}

SensorUser sensorUser = {
    .name = "MyApp",
    .callback = SensorDataCallback,
    .userData = NULL
};

int32_t ret = SubscribeSensor(SENSOR_TYPE_ID_ACCELEROMETER, &sensorUser);
```

**证据**: `interfaces/kits/native/include/sensor_agent.h:71`

---

### UnsubscribeSensor

```c
int32_t UnsubscribeSensor(int32_t sensorTypeId, SensorUser *user);
```

**功能**: 取消订阅传感器数据

**参数**:
- `sensorTypeId` - 传感器类型 ID
- `user` - 传感器用户信息

**返回值**:
- `SENSOR_OK` (0) - 成功
- `SENSOR_ERROR_INVALID_ID` - 无效的传感器 ID
- `SENSOR_ERROR_INVALID_PARAM` - 无效参数

**IPC funcId**: 5

**证据**: `interfaces/kits/native/include/sensor_agent.h:83`

---

### SetMode

```c
int32_t SetMode(int32_t sensorTypeId, SensorUser *user, int32_t mode);
```

**功能**: 设置传感器的数据上报模式

**参数**:
- `sensorTypeId` - 传感器类型 ID
- `user` - 传感器用户信息
- `mode` - 数据上报模式 (`SensorMode`)

**返回值**:
- `SENSOR_OK` (0) - 成功
- `SENSOR_ERROR_INVALID_ID` - 无效的传感器 ID
- `SENSOR_ERROR_INVALID_PARAM` - 无效参数

**IPC funcId**: 6

**证据**: `interfaces/kits/native/include/sensor_agent.h:135`

**注意**: 当前实现中，仅在 `sensor_service_impl.c` 中进行了参数校验，未实际调用底层 HDI 设置模式。

---

### SetOption

```c
int32_t SetOption(int32_t sensorTypeId, SensorUser *user, int32_t option);
```

**功能**: 设置传感器的特殊选项

**参数**:
- `sensorTypeId` - 传感器类型 ID
- `user` - 传感器用户信息
- `option` - 选项值

**返回值**:
- `SENSOR_OK` (0) - 成功
- `SENSOR_ERROR_INVALID_ID` - 无效的传感器 ID
- `SENSOR_ERROR_INVALID_PARAM` - 无效参数

**IPC funcId**: 7

**证据**: `frameworks/src/sensor_agent.c:76-82`

---

## 数据类型

### SensorInfo

```c
typedef struct SensorInfo {
    char sensorName[SENSOR_NAME_MAX_LEN];      // 传感器名称
    char vendorName[SENSOR_NAME_MAX_LEN];       // 厂商名称
    char firmwareVersion[VERSION_MAX_LEN];      // 固件版本
    char hardwareVersion[VERSION_MAX_LEN];     // 硬件版本
    int32_t sensorTypeId;                       // 传感器类型 ID
    int32_t sensorId;                           // 传感器 ID
    float maxRange;                             // 最大量程
    float precision;                            // 精度
    float power;                                // 功耗
} SensorInfo;
```

### SensorEvent

```c
typedef struct SensorEvent {
    int32_t sensorTypeId;       // 传感器类型 ID
    int32_t version;             // 算法版本
    int64_t timestamp;           // 时间戳
    uint32_t option;             // 数据选项
    int32_t mode;                // 上报模式
    uint8_t *data;              // 传感器数据
    uint32_t dataLen;           // 数据长度
} SensorEvent;
```

### SensorUser

```c
typedef struct SensorUser {
    char name[SENSOR_NAME_MAX_LEN];           // 用户名称
    RecordSensorCallback callback;             // 数据回调函数
    UserData *userData;                       // 用户数据
} SensorUser;
```

### UserData

```c
typedef struct UserData {
    char userData[SENSOR_USER_DATA_SIZE];     // 预留用户数据空间
} UserData;
```

---

## 错误码参考

| 错误码 | 值 | 说明 |
|-------|-----|------|
| `SENSOR_OK` | 0 | 成功 |
| `SENSOR_ERROR_UNKNOWN` | -1 | 未知错误 |
| `SENSOR_ERROR_INVALID_ID` | -2 | 无效的传感器 ID |
| `SENSOR_ERROR_INVALID_PARAM` | -3 | 无效参数 |

**证据**: `interfaces/kits/native/include/sensor_agent_type.h:48-51`

---

## 完整使用流程

```
┌─────────────────────────────────────────────────────────────────────┐
│  1. 获取传感器列表                                                   │
│     GetAllSensors(&sensorInfo, &count)                              │
│     ↓                                                               │
│  2. 创建 SensorUser（设置回调函数）                                   │
│     sensorUser.callback = SensorDataCallback                         │
│     ↓                                                               │
│  3. 使能传感器                                                       │
│     ActivateSensor(sensorTypeId, &sensorUser)                       │
│     ↓                                                               │
│  4. 订阅数据                                                         │
│     SubscribeSensor(sensorTypeId, &sensorUser)                      │
│     ↓                                                               │
│  5. 在回调中处理数据                                                 │
│     void SensorDataCallback(SensorEvent *event)                     │
│     ↓                                                               │
│  6. 取消订阅                                                         │
│     UnsubscribeSensor(sensorTypeId, &sensorUser)                    │
│     ↓                                                               │
│  7. 去使能传感器                                                      │
│     DeactivateSensor(sensorTypeId, &sensorUser)                      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## 相关文档

- [项目概览](01_Overview.md)
- [架构设计](03_Architecture.md)
- [安全评审](05_Security.md)
