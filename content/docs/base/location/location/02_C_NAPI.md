# C/N-API 接口参考

> **目的**: 详细说明 C API 和 NDK 接口的函数签名、参数、返回值和使用方法  
> **适用范围**: Native 应用开发者、C/C++ 开发者、使用 NDK 进行定位开发的人员  
> **最后更新**: 2026-02-05

---

## 1. 概述

位置服务提供 C API 接口，支持 Native 应用直接调用定位能力。API 分为以下几类：

| 分类 | 头文件 | 库文件 |
|------|--------|--------|
| 核心定位 | `oh_location.h` | `libohlocation.so` |
| 类型定义 | `oh_location_type.h` | - |

---

## 2. 错误码定义

### 2.1 错误码枚举

```c
// 文件: interfaces/c_api/include/oh_location_type.h:48-76
typedef enum Location_ResultCode {
    LOCATION_SUCCESS = 0,              // 操作成功
    LOCATION_PERMISSION_DENIED = 201,  // 权限拒绝
    LOCATION_INVALID_PARAM = 401,      // 参数错误
    LOCATION_NOT_SUPPORTED = 801,      // 能力不支持
    LOCATION_SERVICE_UNAVAILABLE = 3301000,  // 服务不可用
    LOCATION_SWITCH_OFF = 3301100      // 定位开关关闭
} Location_ResultCode;
```

### 2.2 错误处理建议

| 错误码 | 含义 | 建议处理 |
|--------|------|----------|
| 0 | 成功 | 正常流程 |
| 201 | 权限拒绝 | 请求用户授权 |
| 401 | 参数错误 | 检查 API 参数 |
| 801 | 能力不支持 | 功能降级 |
| 3301000 | 服务异常 | 检查定位开关，稍后重试 |
| 3301100 | 开关关闭 | 引导用户开启定位 |

---

## 3. 类型定义

### 3.1 使用场景枚举

```c
// 文件: interfaces/c_api/include/oh_location_type.h:83-114
typedef enum Location_UseScene {
    LOCATION_USE_SCENE_NAVIGATION = 0x0401,   // 导航场景
    LOCATION_USE_SCENE_SPORT = 0x0402,        // 运动场景
    LOCATION_USE_SCENE_TRANSPORT = 0x0403,    // 出行场景
    LOCATION_USE_SCENE_DAILY_LIFE_SERVICE = 0x0404  // 日常生活服务
} Location_UseScene;
```

**说明**:
- `NAVIGATION`: 室外导航，需要实时位置，GNSS 定位，高功耗
- `SPORT`: 运动轨迹记录，GNSS 定位，高功耗
- `TRANSPORT`: 打车、公共交通，GNSS 定位，高功耗
- `DAILY_LIFE_SERVICE`: 新闻、购物、外卖，仅网络定位，低功耗

### 3.2 功耗场景枚举

```c
// 文件: interfaces/c_api/include/oh_location_type.h:121-145
typedef enum Location_PowerConsumptionScene {
    LOCATION_HIGH_POWER_CONSUMPTION = 0x0601,    // 高功耗
    LOCATION_LOW_POWER_CONSUMPTION = 0x0602,     // 低功耗
    LOCATION_NO_POWER_CONSUMPTION = 0x0603       // 无功耗
} Location_PowerConsumptionScene;
```

**说明**:
- `HIGH_POWER_CONSUMPTION`: 主要使用 GNSS，30秒无结果时切换到网络定位
- `LOW_POWER_CONSUMPTION`: 仅使用网络定位
- `NO_POWER_CONSUMPTION`: 不主动触发定位，仅在其他应用定位时共享

### 3.3 定位源类型枚举

```c
// 文件: interfaces/c_api/include/oh_location_type.h:152-169
typedef enum Location_SourceType {
    LOCATION_SOURCE_TYPE_GNSS = 1,       // GNSS 定位
    LOCATION_SOURCE_TYPE_NETWORK = 2,    // 网络定位
    LOCATION_SOURCE_TYPE_INDOOR = 3,     // 室内定位
    LOCATION_SOURCE_TYPE_RTK = 4         // RTK 高精度定位
} Location_SourceType;
```

### 3.4 位置信息结构体

```c
// 文件: interfaces/c_api/include/oh_location_type.h:176-233
typedef struct Location_BasicInfo {
    double latitude;           // 纬度，-90 到 90
    double longitude;          // 经度，-180 到 180
    double altitude;           // 海拔高度（米）
    double accuracy;           // 水平精度（米）
    double speed;              // 速度（米/秒）
    double direction;          // 方向，0 到 360 度
    int64_t timeForFix;        // 定位时间戳（Unix 毫秒）
    int64_t timeSinceBoot;     // 系统启动时间（纳秒）
    double altitudeAccuracy;   // 垂直精度（米）
    double speedAccuracy;      // 速度精度（米/秒）
    double directionAccuracy;  // 方向精度（度）
    int64_t uncertaintyOfTimeSinceBoot;  // 时间不确定度
    Location_SourceType locationSourceType;  // 定位源类型
} Location_BasicInfo;
```

---

## 4. 核心 API

### 4.1 检查定位开关状态

```c
// 文件: interfaces/c_api/include/oh_location.h:55
Location_ResultCode OH_Location_IsLocatingEnabled(bool* enabled);
```

**功能**: 检查设备定位开关是否开启

**参数**:
- `enabled` [out] - 指向布尔值的指针，用于接收定位开关状态
  - `true`: 定位开关已开启
  - `false`: 定位开关已关闭

**返回值**:
- `LOCATION_SUCCESS`: 成功获取开关状态
- `LOCATION_INVALID_PARAM`: 参数为空指针
- `LOCATION_SERVICE_UNAVAILABLE`: 定位服务异常

**示例**:
```c
#include "oh_location.h"

bool isEnabled = false;
Location_ResultCode result = OH_Location_IsLocatingEnabled(&isEnabled);
if (result == LOCATION_SUCCESS) {
    if (isEnabled) {
        // 定位开关已开启，可以发起定位请求
    } else {
        // 引导用户开启定位开关
    }
}
```

---

### 4.2 开始定位

```c
// 文件: interfaces/c_api/include/oh_location.h:76
Location_ResultCode OH_Location_StartLocating(const Location_RequestConfig* requestConfig);
```

**功能**: 开始定位并订阅位置更新

**参数**:
- `requestConfig` [in] - 指向定位请求配置的指针
  - 使用 `OH_Location_CreateRequestConfig()` 创建
  - 使用 `OH_LocationRequestConfig_SetCallback()` 设置回调
  - 使用 `OH_LocationRequestConfig_SetInterval()` 设置上报间隔

**返回值**:
- `LOCATION_SUCCESS`: 成功开始定位
- `LOCATION_INVALID_PARAM`: 参数为空指针
- `LOCATION_PERMISSION_DENIED`: 权限不足
- `LOCATION_NOT_SUPPORTED`: 能力不支持
- `LOCATION_SERVICE_UNAVAILABLE`: 定位服务异常
- `LOCATION_SWITCH_OFF`: 定位开关关闭

**权限要求**:
- `ohos.permission.APPROXIMATELY_LOCATION`

**前置条件**:
1. 定位开关已开启（`OH_Location_IsLocatingEnabled()` 返回 true）
2. 已获得用户授权
3. 已设置回调函数

---

### 4.3 停止定位

```c
// 文件: interfaces/c_api/include/oh_location.h:99
Location_ResultCode OH_Location_StopLocating(const Location_RequestConfig* requestConfig);
```

**功能**: 停止定位并取消订阅位置更新

**参数**:
- `requestConfig` [in] - 指向定位请求配置的指针
  - 必须与 `OH_Location_StartLocating()` 中传入的指针相同

**返回值**:
- `LOCATION_SUCCESS`: 成功停止定位
- `LOCATION_INVALID_PARAM`: 空指针或与开始时不匹配
- `LOCATION_PERMISSION_DENIED`: 权限不足
- `LOCATION_NOT_SUPPORTED`: 能力不支持
- `LOCATION_SERVICE_UNAVAILABLE`: 定位服务异常
- `LOCATION_SWITCH_OFF`: 定位开关关闭

---

## 5. 配置 API

### 5.1 创建请求配置

```c
// 文件: interfaces/c_api/include/oh_location_type.h:300
Location_RequestConfig* OH_Location_CreateRequestConfig(void);
```

**功能**: 创建定位请求配置实例

**返回值**:
- 非 NULL: 指向配置的指针
- NULL: 创建失败（应用地址空间已满）

**注意**: 使用完成后必须调用 `OH_Location_DestroyRequestConfig()` 释放

---

### 5.2 销毁请求配置

```c
// 文件: interfaces/c_api/include/oh_location_type.h:309
void OH_Location_DestroyRequestConfig(Location_RequestConfig* requestConfig);
```

**功能**: 销毁定位请求配置实例并释放内存

**参数**:
- `requestConfig` [in] - 要销毁的配置指针

---

### 5.3 设置使用场景

```c
// 文件: interfaces/c_api/include/oh_location_type.h:327-328
void OH_LocationRequestConfig_SetUseScene(Location_RequestConfig* requestConfig,
    Location_UseScene useScene);
```

**功能**: 设置定位使用场景

**参数**:
- `requestConfig` - 定位请求配置
- `useScene` - 使用场景枚举值

**优先级**: `useScene` 优先级高于 `powerConsumptionScene`

---

### 5.4 设置功耗场景

```c
// 文件: interfaces/c_api/include/oh_location_type.h:340-341
void OH_LocationRequestConfig_SetPowerConsumptionScene(Location_RequestConfig* requestConfig,
    Location_PowerConsumptionScene powerConsumptionScene);
```

**功能**: 设置功耗场景

**参数**:
- `requestConfig` - 定位请求配置
- `powerConsumptionScene` - 功耗场景枚举值

**注意**: 仅在 `useScene` 未设置时生效

---

### 5.5 设置上报间隔

```c
// 文件: interfaces/c_api/include/oh_location_type.h:352-353
void OH_LocationRequestConfig_SetInterval(Location_RequestConfig* requestConfig,
    int interval);
```

**功能**: 设置位置上报间隔

**参数**:
- `requestConfig` - 定位请求配置
- `interval` - 上报间隔（秒），必须 >= 1，默认值为 1

---

### 5.6 设置回调函数

```c
// 文件: interfaces/c_api/include/oh_location_type.h:366-367
void OH_LocationRequestConfig_SetCallback(Location_RequestConfig* requestConfig,
    Location_InfoCallback callback, void* userData);
```

**功能**: 设置接收位置信息的回调函数

**参数**:
- `requestConfig` - 定位请求配置
- `callback` - 回调函数指针，类型为 `Location_InfoCallback`
- `userData` - 应用数据指针，会在回调中传回

**回调类型定义**:
```c
// 文件: interfaces/c_api/include/oh_location_type.h:283
typedef void (*Location_InfoCallback)(Location_Info* location, void* userData);
```

---

## 6. 位置信息获取 API

### 6.1 获取基本信息

```c
// 文件: interfaces/c_api/include/oh_location_type.h:250
Location_BasicInfo OH_LocationInfo_GetBasicInfo(Location_Info* location);
```

**功能**: 从位置信息结构中获取基本位置数据

**参数**:
- `location` [in] - 指向位置信息结构的指针

**返回值**: `Location_BasicInfo` 结构体，包含位置基本信息

---

### 6.2 获取附加信息

```c
// 文件: interfaces/c_api/include/oh_location_type.h:270-271
Location_ResultCode OH_LocationInfo_GetAdditionalInfo(Location_Info* location,
    char* additionalInfo, uint32_t length);
```

**功能**: 获取位置的附加信息（JSON 格式）

**参数**:
- `location` [in] - 指向位置信息结构的指针
- `additionalInfo` [out] - 存储附加信息的字符串缓冲区
- `length` [in] - 缓冲区大小，建议 >= 256 字节

**返回值**:
- `LOCATION_SUCCESS`: 成功获取
- `LOCATION_INVALID_PARAM`: 空指针或缓冲区过小

---

## 7. 完整使用示例

```c
#include "oh_location.h"
#include <stdio.h>
#include <stdbool.h>

// 回调函数
void LocationCallback(Location_Info* location, void* userData)
{
    Location_BasicInfo basicInfo = OH_LocationInfo_GetBasicInfo(location);
    
    printf("Location received:\n");
    printf("  Latitude: %.6f\n", basicInfo.latitude);
    printf("  Longitude: %.6f\n", basicInfo.longitude);
    printf("  Accuracy: %.2f m\n", basicInfo.accuracy);
    printf("  Source: %d\n", basicInfo.locationSourceType);
}

int main()
{
    // 1. 检查定位开关
    bool isEnabled = false;
    Location_ResultCode result = OH_Location_IsLocatingEnabled(&isEnabled);
    if (result != LOCATION_SUCCESS || !isEnabled) {
        printf("Location is disabled or unavailable\n");
        return -1;
    }
    
    // 2. 创建请求配置
    Location_RequestConfig* config = OH_Location_CreateRequestConfig();
    if (config == NULL) {
        printf("Failed to create request config\n");
        return -1;
    }
    
    // 3. 设置参数
    OH_LocationRequestConfig_SetUseScene(config, LOCATION_USE_SCENE_DAILY_LIFE_SERVICE);
    OH_LocationRequestConfig_SetInterval(config, 5);  // 5秒上报一次
    OH_LocationRequestConfig_SetCallback(config, LocationCallback, NULL);
    
    // 4. 开始定位
    result = OH_Location_StartLocating(config);
    if (result != LOCATION_SUCCESS) {
        printf("Failed to start locating: %d\n", result);
        OH_Location_DestroyRequestConfig(config);
        return -1;
    }
    
    // 5. 业务逻辑...
    
    // 6. 停止定位
    OH_Location_StopLocating(config);
    
    // 7. 释放资源
    OH_Location_DestroyRequestConfig(config);
    
    return 0;
}
```

---

## 8. API 变更记录

| 版本 | 变更 |
|------|------|
| 13 | 初始版本 |

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [概览](index.md) | 项目定位与核心能力 |
| [系统架构](01_Architecture.md) | 详细架构说明 |
| [JS API 接口](03_JS_API.md) | JS API 参考 |
| [编译产物](06_Artifacts.md) | 库文件说明 |

---

## 更新日志

| 日期 | 版本 | 变更 |
|------|------|------|
| 2026-02-05 | 1.0 | 初始版本 |
