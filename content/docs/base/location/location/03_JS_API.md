# JS API 接口参考

> **目的**: 详细说明 JS API 的模块结构、类定义、方法签名和使用示例  
> **适用范围**: JS/ETS 应用开发者、使用 @ohos.geolocation 模块的开发人员  
> **最后更新**: 2026-02-05

---

## 1. 概述

位置服务的 JS API 通过 `@ohos.geolocation` 模块提供，支持以下能力：

| 能力 | 模块 | 说明 |
|------|------|------|
| 定位 | `@ohos.geolocation` | 获取位置、订阅位置更新 |
| 地理围栏 | `@ohos.geolocation.geofence` | 地理围栏管理 |
| 坐标转换 | `@ohos.geolocation` | WGS84/GCJ02 坐标转换 |

> **官方 API 参考**: https://gitee.com/openharmony/interface_sdk-js/blob/master/api/@ohos.geolocation.d.ts

---

## 2. 主要类与命名空间

### 2.1 Geolocation 类

`@ohos.geolocation.Geolocation` 提供核心定位功能：

```typescript
// 能力检查
static isGeolocationEnabled(): boolean;

// 单次定位
static getCurrentLocation(request: LocationRequest): Promise<Location>;
static getCurrentLocation(request: LocationRequest, callback: AsyncCallback<Location>);

// 持续定位
static subscribeLocationCallback(request: LocationRequest, callback: AsyncCallback<Location>): void;
static unsubscribeLocationCallback(callback: AsyncCallback<Location>): void;

// 附近设备定位
static getAddressesFromLocation(request: ReverseGeocodeRequest): Promise<Array<GeoAddress>>;
static getAddressesFromLocation(request: ReverseGeocodeRequest, callback: AsyncCallback<Array<GeoAddress>>): void;
```

### 2.2 LocationRequest 接口

定位请求参数：

```typescript
interface LocationRequest {
    priority: LocationRequestPriority;  // 优先级
    scenario: LocationRequestScenario;   // 场景
    timeInterval?: number;               // 时间间隔（秒）
    distanceInterval?: number;           // 距离间隔（米）
}
```

### 2.3 LocationRequestPriority 枚举

```typescript
enum LocationRequestPriority {
    PRIORITY_UNSET = 0x200,           // 未设置
    PRIORITY_ACCURACY = 0x201,        // 精度优先
    PRIORITY_LOW_POWER = 0x202,       // 低功耗优先
    PRIORITY_FIRST_FIX = 0x203,       // 首次定位优先
}
```

### 2.4 LocationRequestScenario 枚举

```typescript
enum LocationRequestScenario {
    SCENE_UNSET = 0x400,              // 未设置
    SCENE_NAVIGATION = 0x401,         // 导航
    SCENE_SPORT = 0x402,              // 运动
    SCENE_TRANSPORT = 0x403,          // 出行
    SCENE_DAILY_LIFE_SERVICE = 0x404, // 日常生活服务
}
```

### 2.5 Location 接口

位置信息：

```typescript
interface Location {
    latitude: number;                 // 纬度
    longitude: number;                // 经度
    altitude: number;                 // 海拔
    accuracy: number;                 // 精度
    speed: number;                    // 速度
    direction: number;                // 方向
    timeForFix: number;               // 定位时间戳
    timeSinceBoot: number;            // 系统启动时间
    additions?: Map<string, string>;  // 附加信息
    additionVaild?: boolean;          // 附加信息有效性
}
```

---

## 3. LocationManager

位置管理器提供更多高级功能：

```typescript
import geolocation from '@ohos.geolocation';

// 获取 LocationManager
let locationManager = geolocation.getLocationManager();
```

### 3.1 主要方法

```typescript
// 检查定位开关
static isLocationEnabled(): boolean;

// 启用定位开关
static enableLocation(): void;

// 禁用定位开关
static disableLocation(): void;

// 订阅开关状态变化
static on('locationStateChange', callback: AsyncCallback<boolean>): void;

// 单次定位
getCurrentLocation(request: LocationRequest): Promise<Location>;
getCurrentLocation(request: LocationRequest, callback: AsyncCallback<Location>): void;

// 持续定位
subscribeLocationCallback(request: LocationRequest, callback: AsyncCallback<Location>): void;
unsubscribeLocationCallback(callback: AsyncCallback<Location>): void;
```

---

## 4. 使用示例

### 4.1 检查定位开关

```typescript
import geolocation from '@ohos.geolocation';

let isEnabled = geolocation.isLocationEnabled();
console.log('Location enabled:', isEnabled);
```

### 4.2 获取当前位置

```typescript
import geolocation from '@ohos.geolocation';

let request = {
    priority: geolocation.LocationRequestPriority.PRIORITY_ACCURACY,
    scenario: geolocation.LocationRequestScenario.SCENE_NAVIGATION,
    timeInterval: 0,
    distanceInterval: 0
};

geolocation.getCurrentLocation(request).then((location) => {
    console.log('Latitude:', location.latitude);
    console.log('Longitude:', location.longitude);
    console.log('Accuracy:', location.accuracy);
}).catch((error) => {
    console.error('Failed to get location:', error);
});
```

### 4.3 持续定位

```typescript
import geolocation from '@ohos.geolocation';

let request = {
    priority: geolocation.LocationRequestPriority.PRIORITY_ACCURACY,
    scenario: geolocation.LocationRequestScenario.SCENE_SPORT,
    timeInterval: 5,  // 5秒上报一次
    distanceInterval: 0
};

let locationCallback = (location) => {
    console.log('New location:', location.latitude, location.longitude);
};

// 订阅位置更新
geolocation.subscribeLocationCallback(request, locationCallback);

// 业务逻辑...

// 取消订阅
geolocation.unsubscribeLocationCallback(locationCallback);
```

### 4.4 订阅开关状态变化

```typescript
import geolocation from '@ohos.geolocation';

geolocation.on('locationStateChange', (state) => {
    console.log('Location state changed:', state ? 'enabled' : 'disabled');
});
```

---

## 5. 地理编码

### 5.1 逆地理编码（坐标 → 地址）

```typescript
import geolocation from '@ohos.geolocation';

let request = {
    location: {
        latitude: 39.9,  // 北京
        longitude: 116.4
    },
    maxItems: 1
};

geolocation.getAddressesFromLocation(request).then((addresses) => {
    if (addresses.length > 0) {
        console.log('Address:', addresses[0].placeName);
    }
}).catch((error) => {
    console.error('Failed to get address:', error);
});
```

### 5.2 地理编码（地址 → 坐标）

```typescript
import geolocation from '@ohos.geolocation';

let request = {
    description: '北京市海淀区西二旗',
    maxItems: 1
};

geolocation.getAddressesFromDescription(request).then((addresses) => {
    if (addresses.length > 0) {
        console.log('Latitude:', addresses[0].latitude);
        console.log('Longitude:', addresses[0].longitude);
    }
}).catch((error) => {
    console.error('Failed to geocode:', error);
});
```

---

## 6. 权限申请

### 6.1 声明权限

在 `module.json5` 中声明：

```json
{
  "module": {
    "requestPermissions": [
      {
        "name": "ohos.permission.LOCATION",
        "usedScene": {
          "when": "inuse"
        }
      },
      {
        "name": "ohos.permission.APPROXIMATELY_LOCATION",
        "usedScene": {
          "when": "inuse"
        }
      }
    ]
  }
}
```

### 6.2 运行时请求

```typescript
import abilityAccessCtrl from '@ohos.abilityAccessCtrl';
import bundleManager from '@ohos.bundle.bundleManager';

let atManager = abilityAccessCtrl.createAtManager();
let bundleInfo = bundleManager.getBundleInfoForSelfSync(
    bundleManager.BundleFlag.GET_BUNDLE_INFO_WITH_APPLICATION
);
let tokenId = bundleInfo.appInfo.accessTokenId;

atManager.requestPermissionFromUser(
    ['ohos.permission.LOCATION'],
    (error, result) => {
        if (error) {
            console.error('Request permission error:', error);
            return;
        }
        console.log('Permission result:', result);
    }
);
```

---

## 7. 错误码

| 错误码 | 说明 |
|--------|------|
| 401 | 参数错误 |
| 801 | 能力不支持 |
| 3301000 | 定位服务不可用 |
| 3301100 | 定位开关关闭 |

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [概览](index.md) | 项目定位与核心能力 |
| [系统架构](01_Architecture.md) | 详细架构说明 |
| [C/N-API 接口](02_C_NAPI.md) | C API 参考 |
| [安全风险评审](07_Security.md) | 安全分析 |

---

## 更新日志

| 日期 | 版本 | 变更 |
|------|------|------|
| 2026-02-05 | 1.0 | 初始版本 |
