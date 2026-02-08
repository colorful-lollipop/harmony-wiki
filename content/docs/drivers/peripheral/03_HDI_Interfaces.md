# HDI 接口说明

本文档详细说明各模块的 HDI（Hardware Driver Interface）接口定义。

## 3.1 接口设计原则

### 1. C 语言结构体函数指针模式
```c
// 定义接口结构体
struct AudioManager {
    int32_t (*GetAllAdapters)(struct AudioManager *manager,
                              struct AudioAdapterDescriptor **descs,
                              int32_t *size);
    int32_t (*LoadAdapter)(struct AudioManager *manager,
                           const struct AudioAdapterDescriptor *desc,
                           struct AudioAdapter **adapter);
    void (*UnloadAdapter)(struct AudioManager *manager,
                          struct AudioAdapter *adapter);
};
```

### 2. 返回值规范
- `int32_t`：0 表示成功，负数表示错误
- 输出参数使用指针传递
- 字符串使用 `char *` 传递

### 3. 版本化策略
- 主版本号表示不兼容变更（v2.0）
- 次版本号表示向后兼容（v1.1）
- 接口位置：`interfaces/v1_0/`, `interfaces/2.0/`

---

## 3.2 Audio 模块 HDI 接口

### 3.2.1 AudioManager 接口

**头文件**: `audio/interfaces/include/audio_manager.h`

| 接口名 | 参数 | 返回值 | 功能描述 |
|--------|------|--------|----------|
| `GetAllAdapters` | manager, descs, size | int32_t | 获取所有音频适配器列表 |
| `LoadAdapter` | manager, desc, adapter | int32_t | 加载音频适配器 |
| `UnloadAdapter` | manager, adapter | void | 卸载音频适配器 |

**函数指针签名**:
```c
int32_t (*GetAllAdapters)(struct AudioManager *manager,
                          struct AudioAdapterDescriptor **descs,
                          int32_t *size);
```

**证据**: `audio/interfaces/include/audio_manager.h:15-30`

---

### 3.2.2 AudioAdapter 接口

**头文件**: `audio/interfaces/include/audio_adapter.h`

| 接口名 | 参数 | 返回值 | 功能描述 |
|--------|------|--------|----------|
| `InitAllPorts` | adapter | int32_t | 初始化所有端口 |
| `CreateRender` | adapter, desc, attrs, render | int32_t | 创建音频渲染器 |
| `DestroyRender` | adapter, render | int32_t | 销毁音频渲染器 |
| `CreateCapture` | adapter, desc, attrs, capture | int32_t | 创建音频采集器 |
| `DestroyCapture` | adapter, capture | int32_t | 销毁音频采集器 |
| `GetPortCapability` | adapter, port, capability | int32_t | 获取端口能力 |
| `SetPassthroughMode` | adapter, port, mode | int32_t | 设置透传模式 |
| `GetPassthroughMode` | adapter, port, mode | int32_t | 获取透传模式 |

**证据**: `audio/interfaces/include/audio_adapter.h`

---

### 3.2.3 AudioRender 接口

**头文件**: `audio/interfaces/include/audio_render.h`

| 接口名 | 参数 | 返回值 | 功能描述 |
|--------|------|--------|----------|
| `RenderFrame` | render, frame, requestBytes, replyBytes | int32_t | 播放音频帧 |
| `GetRenderPosition` | render, frames, time | int32_t | 获取播放位置 |
| `SetRenderSpeed` | render, speed | int32_t | 设置播放速度 |
| `GetRenderSpeed` | render, speed | int32_t | 获取播放速度 |
| `SetChannelMode` | render, mode | int32_t | 设置通道模式 |
| `GetChannelMode` | render, mode | int32_t | 获取通道模式 |
| `SetVolume` | render, volume | int32_t | 设置音量 |
| `GetVolume` | render, volume | int32_t | 获取音量 |

**证据**: `audio/interfaces/include/audio_render.h`

---

## 3.3 Input 模块 HDI 接口

### 3.3.1 InputManager 接口

**头文件**: `input/interfaces/include/input_manager.h`

| 接口名 | 参数 | 返回值 | 功能描述 | 证据位置 |
|--------|------|--------|----------|----------|
| `OpenInputDevice` | devIndex | int32_t | 打开输入设备 | `input_manager.h:10` |
| `CloseInputDevice` | devIndex | int32_t | 关闭输入设备 | `input_manager.h:11` |
| `GetInputDevice` | devIndex, devInfo | int32_t | 获取指定设备信息 | `input_manager.h:12` |
| `GetInputDeviceList` | devNum, devList, size | int32_t | 获取设备列表 | `input_manager.h:13` |

**结构体定义**:
```c
struct IInputInterface {
    struct IInputManager *iInputManager;
    struct IInputReporter *iInputReporter;
    struct IInputController *iInputController;
};

struct IInputManager {
    int32_t (*OpenInputDevice)(uint32_t devIndex);
    int32_t (*CloseInputDevice)(uint32_t devIndex);
    int32_t (*GetInputDevice)(uint32_t devIndex, DeviceInfo **devInfo);
    int32_t (*GetInputDeviceList)(uint32_t *devNum, DeviceInfo **devList, uint32_t size);
};
```

**证据**: `input/interfaces/include/input_manager.h:5-20`

---

### 3.3.2 InputReporter 接口

**头文件**: `input/interfaces/include/input_reporter.h`

| 接口名 | 参数 | 返回值 | 功能描述 | 证据位置 |
|--------|------|--------|----------|----------|
| `RegisterReportCallback` | devIndex, callback | int32_t | 注册事件回调 | `input_reporter.h:8` |
| `UnregisterReportCallback` | devIndex | int32_t | 注销事件回调 | `input_reporter.h:9` |

**回调函数类型**:
```c
typedef void (*ReportEventPkgCallback)(const EventPackage **pkgs, uint32_t count);

struct InputReportEventCb {
    ReportEventPkgCallback ReportEventPkgCallback;
};
```

**证据**: `input/interfaces/include/input_reporter.h:5-15`

---

### 3.3.3 InputController 接口

**头文件**: `input/interfaces/include/input_controller.h`

| 接口名 | 参数 | 返回值 | 功能描述 | 证据位置 |
|--------|------|--------|----------|----------|
| `SetPowerStatus` | devIndex, status | int32_t | 设置电源状态 | `input_controller.h:10` |
| `GetPowerStatus` | devIndex, status | int32_t | 获取电源状态 | `input_controller.h:11` |
| `GetDeviceType` | devIndex, deviceType | int32_t | 获取设备类型 | `input_controller.h:12` |
| `GetChipInfo` | devIndex, chipInfo, length | int32_t | 获取芯片信息 | `input_controller.h:13` |
| `GetVendorName` | devIndex, vendorName, length | int32_t | 获取厂商名 | `input_controller.h:14` |
| `GetChipName` | devIndex, chipName, length | int32_t | 获取芯片名 | `input_controller.h:15` |
| `SetGestureMode` | devIndex, gestureMode | int32_t | 设置手势模式 | `input_controller.h:16` |
| `RunCapacitanceTest` | devIndex, testType, result, length | int32_t | 电容自检 | `input_controller.h:17` |
| `RunExtraCommand` | devIndex, cmd | int32_t | 执行扩展命令 | `input_controller.h:18` |

**证据**: `input/interfaces/include/input_controller.h:5-25`

---

## 3.4 Sensor 模块 HDI 接口

### 3.4.1 SensorInterface 接口

**头文件**: `sensor/interfaces/include/sensor_if.h`

| 接口分类 | 接口名 | 参数 | 返回值 | 功能描述 | 证据位置 |
|----------|--------|------|--------|----------|----------|
| **查询** | `GetAllSensors` | sensorInfo, count | int32_t | 获取所有传感器 | `sensor_if.h` |
| **配置** | `Enable` | sensorId | int32_t | 使能传感器 | `sensor_if.h` |
| **配置** | `Disable` | sensorId | int32_t | 去使能传感器 | `sensor_if.h` |
| **配置** | `SetBatch` | sensorId, samplingInterval, reportInterval | int32_t | 设置采样参数 | `sensor_if.h` |
| **配置** | `SetMode` | sensorTypeId, user, mode | int32_t | 设置工作模式 | `sensor_if.h` |
| **配置** | `SetOption` | sensorId, option | int32_t | 设置可选配置 | `sensor_if.h` |
| **订阅** | `Register` | user, cb | int32_t | 注册数据回调 | `sensor_if.h` |
| **订阅** | `Unregister` | user | int32_t | 取消订阅 | `sensor_if.h` |
| **实例** | `NewSensorInterfaceInstance` | void | SensorInterface* | 创建接口实例 | `sensor_if.h` |
| **实例** | `FreeSensorInterfaceInstance` | void | int32_t | 释放接口实例 | `sensor_if.h` |

**结构体定义**:
```c
struct SensorInterface {
    int32_t (*GetAllSensors)(struct SensorInformation **sensorInfo, int32_t *count);
    int32_t (*Enable)(int32_t sensorId);
    int32_t (*Disable)(int32_t sensorId);
    int32_t (*SetBatch)(int32_t sensorId, int64_t samplingInterval, int64_t reportInterval);
    int32_t (*SetMode)(int32_t sensorTypeId, struct SensorUser *user, int32_t mode);
    int32_t (*SetOption)(int32_t sensorId, uint32_t option);
    int32_t (*Register)(struct SensorUser *user, RecordDataCallback cb);
    int32_t (*Unregister)(struct SensorUser *user);
};
```

**证据**: `sensor/interfaces/include/sensor_if.h`, `sensor/README_zh.md:54-109`

---

## 3.5 通用接口模式

### 3.5.1 回调注册模式

**Input 模块示例**:
```c
// 1. 定义回调函数
static void ReportEventPkgCallback(const EventPackage **pkgs, uint32_t count)
{
    // 处理输入事件
}

// 2. 初始化回调结构
InputReportEventCb callback;
callback.ReportEventPkgCallback = ReportEventPkgCallback;

// 3. 注册回调
inputInterface->iInputReporter->RegisterReportCallback(devIndex, &callback);
```

**证据**: `input/README_zh.md:167-213`

---

### 3.5.2 异步数据上报模式

**Sensor 模块示例**:
```c
// 1. 定义数据回调
void SensorDataCallback(struct SensorEvents *event)
{
    float *data = (float *)event->data;
    // 处理传感器数据
}

// 2. 创建接口实例
const struct SensorInterface *sensorDev = NewSensorInterfaceInstance();

// 3. 注册回调
sensorDev->Register(user, SensorDataCallback);

// 4. 使能传感器
sensorDev->Enable(SENSOR_TYPE_ACCELEROMETER);

// 5. 数据通过回调自动上报
```

**证据**: `sensor/README_zh.md:121-178`

---

## 3.6 接口使用示例

### Input 模块完整调用流程

```c
#include "input_manager.h"
#include "input_controller.h"
#include "input_reporter.h"

#define DEV_INDEX 1

IInputInterface *g_inputInterface;
InputReportEventCb g_callback;

// 数据上报回调
static void ReportEventPkgCallback(const EventPackage **pkgs, uint32_t count)
{
    for (uint32_t i = 0; i < count; i++) {
        HDF_LOGI("type=%u, code=%u, value=%d",
                  pkgs[i]->type, pkgs[i]->code, pkgs[i]->value);
    }
}

int InputServiceSample(void)
{
    int ret;
    uint32_t devType;

    // 1. 获取接口
    ret = GetInputInterface(&g_inputInterface);
    if (ret != INPUT_SUCCESS) {
        HDF_LOGE("GetInputInterface failed, ret=%d", ret);
        return ret;
    }

    // 2. 打开设备
    ret = g_inputInterface->iInputManager->OpenInputDevice(DEV_INDEX);
    if (ret != INPUT_SUCCESS) {
        HDF_LOGE("OpenInputDevice failed, ret=%d", ret);
        return ret;
    }

    // 3. 获取设备信息
    ret = g_inputInterface->iInputController->GetDeviceType(DEV_INDEX, &devType);
    if (ret != INPUT_SUCCESS) {
        HDF_LOGE("GetDeviceType failed, ret=%d", ret);
        return ret;
    }
    HDF_LOGI("Device type=%u", devType);

    // 4. 注册回调
    g_callback.ReportEventPkgCallback = ReportEventPkgCallback;
    ret = g_inputInterface->iInputReporter->RegisterReportCallback(DEV_INDEX, &g_callback);
    if (ret != INPUT_SUCCESS) {
        HDF_LOGE("RegisterReportCallback failed, ret=%d", ret);
        return ret;
    }

    // 5. 等待事件...

    // 6. 注销回调
    ret = g_inputInterface->iInputReporter->UnregisterReportCallback(DEV_INDEX);
    if (ret != INPUT_SUCCESS) {
        HDF_LOGE("UnregisterReportCallback failed, ret=%d", ret);
        return ret;
    }

    // 7. 关闭设备
    ret = g_inputInterface->iInputManager->CloseInputDevice(DEV_INDEX);
    if (ret != INPUT_SUCCESS) {
        HDF_LOGE("CloseInputDevice failed, ret=%d", ret);
        return ret;
    }

    return INPUT_SUCCESS;
}
```

**证据**: `input/README_zh.md:159-231`

---

## 3.7 接口稳定性标注

### 稳定接口标识
- 稳定接口：位于 `interfaces/` 或 `interfaces/2.0/` 目录
- 实验接口：位于 `interfaces/v1_0-experimental/` 目录（如果有）
- 废弃接口：在注释中标注 `@deprecated`

### 稳定性证据来源
1. 目录层级
2. 头文件注释
3. 接口命名

---

**下一节**: [架构说明](04_Architecture.md) - 了解系统架构细节
