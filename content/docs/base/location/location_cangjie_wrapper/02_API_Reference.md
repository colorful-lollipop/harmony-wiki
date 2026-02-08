# API 参考

> Cangjie 语言位置服务 API 完整参考

## 概述

本文档描述 `ohos.geo_location_manager` 模块提供的 Cangjie API 接口。所有 API 均声明为 Beta 特性，API Level 22+ 可用。

### 包信息

| 属性 | 值 |
|------|-----|
| 包名 | `ohos.geo_location_manager` |
| Kit 包名 | `kit.LocationKit` |
| API Level | 22+ |
| 系统能力 | SystemCapability.Location.Location.Core |

### 使用前提

1. **权限申请**：需要 `ohos.permission.APPROXIMATELY_LOCATION` 权限
2. **用户授权**：用户必须授权位置访问权限
3. **位置开关**：用户必须开启设备位置开关

## GeoLocationManager 类

> 位置管理器，提供获取位置和检查状态的能力

**包**：`ohos.geo_location_manager`  
**代码位置**：`ohos/geo_location_manager/geo_location_manager.cj:32`

### 方法列表

| 方法 | 功能 | 同步/异步 | 工作线程 |
|------|------|----------|----------|
| `getCurrentLocation()` | 获取当前位置（默认配置） | 同步 | 是 |
| `getCurrentLocation(request: CurrentLocationRequest)` | 获取当前位置（指定配置） | 同步 | 是 |
| `getCurrentLocation(request: SingleLocationRequest)` | 获取当前位置（单次请求） | 同步 | 是 |
| `isLocationEnabled()` | 检查位置开关状态 | 同步 | 否 |

#### getCurrentLocation()

获取当前设备的地理位置信息。

**签名**：
```cj
public static func getCurrentLocation(): Location
```

**返回值**：
| 类型 | 说明 |
|------|------|
| `Location` | 当前设备位置信息 |

**异常**：
| 错误码 | 含义 | 说明 |
|--------|------|------|
| 201 | 权限验证失败 | 应用无位置权限 |
| 801 | 能力不支持 | 设备不支持定位 |
| 3301000 | 位置服务不可用 | 服务异常 |
| 3301100 | 位置开关关闭 | 用户未开启位置 |
| 3301200 | 获取位置失败 | GPS 信号弱等 |

**使用示例**：
```cj
import ohos.geo_location_manager.*

let location = GeoLocationManager.getCurrentLocation()
println("纬度: ${location.latitude}")
println("经度: ${location.longitude}")
```

---

#### getCurrentLocation(request: CurrentLocationRequest)

使用指定配置获取当前位置。

**签名**：
```cj
public static func getCurrentLocation(request: CurrentLocationRequest): Location
```

**参数**：
| 参数名 | 类型 | 必填 | 默认值 | 说明 |
|--------|------|------|--------|------|
| request | CurrentLocationRequest | 否 | 优先级 FirstFix | 定位请求配置 |

**返回值**：
| 类型 | 说明 |
|------|------|
| `Location` | 当前设备位置信息 |

**异常**：同 `getCurrentLocation()`

**使用示例**：
```cj
import ohos.geo_location_manager.*

let request = CurrentLocationRequest(
    priority: LocationRequestPriority.Accuracy,
    scenario: LocationRequestScenario.Navigation,
    maxAccuracy: 10.0,
    timeoutMs: 10000
)
let location = GeoLocationManager.getCurrentLocation(request)
```

---

#### getCurrentLocation(request: SingleLocationRequest)

使用单次定位配置获取当前位置。

**签名**：
```cj
public static func getCurrentLocation(request: SingleLocationRequest): Location
```

**参数**：
| 参数名 | 类型 | 必填 | 说明 |
|--------|------|------|------|
| request | SingleLocationRequest | 是 | 单次定位请求配置 |

**返回值**：
| 类型 | 说明 |
|------|------|
| `Location` | 当前设备位置信息 |

**异常**：同 `getCurrentLocation()`

**使用示例**：
```cj
import ohos.geo_location_manager.*

let request = SingleLocationRequest(
    locatingPriority: LocatingPriority.PriorityLocatingSpeed,
    locatingTimeoutMs: 5000
)
let location = GeoLocationManager.getCurrentLocation(request)
```

---

#### isLocationEnabled()

检查设备位置服务开关是否开启。

**签名**：
```cj
public static func isLocationEnabled(): Bool
```

**返回值**：
| 值 | 说明 |
|---|------|
| `true` | 位置开关已开启 |
| `false` | 位置开关未开启 |

**异常**：
| 错误码 | 含义 |
|--------|------|
| 801 | 能力不支持 |
| 3301000 | 位置服务不可用 |

**使用示例**：
```cj
import ohos.geo_location_manager.*

let isEnabled = GeoLocationManager.isLocationEnabled()
if (!isEnabled) {
    println("请开启位置开关")
}
```

---

## Location 类

> 地理位置信息数据类

**包**：`ohos.geo_location_manager`  
**代码位置**：`ohos/geo_location_manager/geo_location_manager_common.cj:109`

### 属性列表

| 属性 | 类型 | 可选 | 说明 |
|------|------|------|------|
| latitude | Float64 | 否 | 纬度，正值为北纬 |
| longitude | Float64 | 否 | 经度，正值为东经 |
| altitude | Float64 | 否 | 海拔高度（米） |
| accuracy | Float64 | 否 | 定位精度（米） |
| speed | Float64 | 否 | 速度（米/秒） |
| timestamp | Int64 | 否 | UTC 时间戳 |
| direction | Float64 | 否 | 方向（度） |
| timeSinceBoot | Int64 | 否 | 设备启动后的时间戳 |
| additions | Array<String>? | 是 | 附加信息 |
| additionsMap | Map<String, String>? | 是 | 附加信息映射 |
| additionSize | Int64? | 是 | 附加信息数量 |
| altitudeAccuracy | Float64? | 是 | 垂直精度（米） |
| speedAccuracy | Float64? | 是 | 速度精度（米/秒） |
| directionAccuracy | Float64? | 是 | 方向精度（度） |
| uncertaintyOfTimeSinceBoot | Int64? | 是 | 时间不确定度（纳秒） |
| sourceType | LocationSourceType? | 是 | 位置数据来源 |

### 使用示例

```cj
import ohos.geo_location_manager.*

let location = GeoLocationManager.getCurrentLocation()

// 基本信息
println("位置: (${location.latitude}, ${location.longitude})")
println("海拔: ${location.altitude} 米")
println("精度: ${location.accuracy} 米")

// 运动信息
println("速度: ${location.speed} 米/秒")
println("方向: ${location.direction} 度")

// 时间信息
println("UTC 时间: ${location.timestamp}")
println("启动后时间: ${location.timeSinceBoot} 纳秒")
```

---

## 枚举类型

### LocationSourceType

> 定位数据来源类型

**代码位置**：`ohos/geo_location_manager/geo_location_manager_common.cj:40`

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| Gnss | 1 | GNSS 卫星定位 |
| Network | 2 | 网络定位 |
| Indoor | 3 | 室内定位 |
| Rtk | 4 | RTK 高精度定位 |

**使用示例**：
```cj
if (let Some(source) <- location.sourceType) {
    match (source) {
        case LocationSourceType.Gnss => println("GPS 定位")
        case LocationSourceType.Network => println("网络定位")
        case _ => println("其他定位")
    }
}
```

---

### LocationRequestPriority

> 定位请求优先级

**代码位置**：`ohos/geo_location_manager/geo_location_manager_common.cj:276`

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| Unset | 0x200 | 未设置 |
| Accuracy | 0x201 | 优先精度 |
| LowPower | 0x201 | 优先低功耗 |
| FirstFix | 0x203 | 优先首次定位速度 |

---

### LocationRequestScenario

> 定位请求场景

**代码位置**：`ohos/geo_location_manager/geo_location_manager_common.cj:332`

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| Unset | 0x300 | 未设置 |
| Navigation | 0x301 | 导航 |
| TrajectoryTracking | 0x302 | 轨迹追踪 |
| CarHailing | 0x303 | 网约车 |
| DailyLifeService | 0x304 | 日常生活 |
| NoPower | 0x305 | 节能 |

---

### LocatingPriority

> 单次定位优先级

**代码位置**：`ohos/geo_location_manager/geo_location_manager_common.cj:471`

| 枚举值 | 值 | 说明 |
|--------|-----|------|
| PriorityAccuracy | 0x501 | 优先精度 |
| PriorityLocatingSpeed | 0x502 | 优先定位速度 |

---

## 请求配置类

### CurrentLocationRequest

> 持续定位请求配置

**代码位置**：`ohos/geo_location_manager/geo_location_manager_common.cj:408`

**构造函数**：
```cj
public init(
    priority!: LocationRequestPriority = LocationRequestPriority.FirstFix,
    scenario!: LocationRequestScenario = LocationRequestScenario.Unset,
    maxAccuracy!: Float32 = 0.0,
    timeoutMs!: Int32 = 5000
)
```

**属性**：
| 属性 | 类型 | 必填 | 默认值 | 说明 |
|------|------|------|--------|------|
| priority | LocationRequestPriority | 否 | FirstFix | 定位优先级 |
| scenario | LocationRequestScenario | 否 | Unset | 定位场景 |
| maxAccuracy | Float32 | 否 | 0.0 | 最大精度要求（米） |
| timeoutMs | Int32 | 否 | 5000 | 超时时间（毫秒） |

**使用示例**：
```cj
let request = CurrentLocationRequest(
    priority: LocationRequestPriority.Accuracy,
    scenario: LocationRequestScenario.Navigation,
    maxAccuracy: 5.0,
    timeoutMs: 15000
)
```

---

### SingleLocationRequest

> 单次定位请求配置

**代码位置**：`ohos/geo_location_manager/geo_location_manager_common.cj:507`

**构造函数**：
```cj
public init(
    locatingPriority: LocatingPriority,
    locatingTimeoutMs: Int32
)
```

**属性**：
| 属性 | 类型 | 必填 | 说明 |
|------|------|------|------|
| locatingPriority | LocatingPriority | 是 | 定位优先级 |
| locatingTimeoutMs | Int32 | 是 | 超时时间（毫秒） |

**使用示例**：
```cj
let request = SingleLocationRequest(
    locatingPriority: LocatingPriority.PriorityLocatingSpeed,
    locatingTimeoutMs: 3000
)
```

---

## 错误码参考

| 错误码 | 错误消息 | 说明 |
|--------|----------|------|
| 201 | Permission verification failed | 权限验证失败 |
| 801 | Capability not supported | 设备能力不支持 |
| 3301000 | The location service is unavailable | 位置服务不可用 |
| 3301100 | The location switch is off | 位置开关关闭 |
| 3301200 | Failed to obtain the geographical location | 获取位置失败 |
| 3301300 | Reverse geocoding query failed | 逆地理编码查询失败 |
| 3301400 | Geocoding query failed | 地理编码查询失败 |

证据来源：`geo_location_manager_common.cj:552-566`

---

## 完整使用示例

```cj
package ohos.app

import ohos.geo_location_manager.*
import ohos.business_exception.BusinessException

func getUserLocation(): Location? {
    try {
        // 检查位置开关
        if (!GeoLocationManager.isLocationEnabled()) {
            println("请开启设备位置开关")
            return None
        }

        // 获取位置
        let request = CurrentLocationRequest(
            priority: LocationRequestPriority.Accuracy,
            scenario: LocationRequestScenario.DailyLifeService,
            timeoutMs: 10000
        )

        return GeoLocationManager.getCurrentLocation(request)
    } catch (e: BusinessException) {
        println("获取位置失败: ${e.code} - ${e.message}")
        return None
    }
}

func main() {
    match (getUserLocation()) {
        case Some(location) => {
            println("纬度: ${location.latitude}")
            println("经度: ${location.longitude}")
            println("精度: ${location.accuracy} 米")
        }
        case None => {
            println("无法获取位置信息")
        }
    }
}
```

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [00_Overview](00_Overview.md) | 项目概览 |
| [01_Architecture](01_Architecture.md) | 系统架构 |
| [03_Build](03_Build.md) | 构建配置 |
| [04_Security](04_Security.md) | 安全注意事项 |

## 外部参考

- [Location 开发指南](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/tree/master/doc/Dev_Guide/source_zh_cn/location/cj-location-guidelines.md)
- [LocationKit API 参考](https://gitcode.com/openharmony-sig/arkcompiler_cangjie_ark_interop/tree/master/doc/API_Reference/source_zh_cn/apis/LocationKit)
