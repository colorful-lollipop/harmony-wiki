# 攻击面分析 (Attack Surface Analysis)

> **目的**: 全面识别位置服务组件的外部输入入口、敏感操作点、信任边界跨越点  
> **适用范围**: 安全研究员、渗透测试人员、安全架构师  
> **最后更新**: 2026-02-07

---

## 1. 威胁模型与信任边界

### 1.1 信任边界图

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                            不可信区域 (Untrusted)                            │
│  ┌─────────────┐  ┌─────────────┐  ┌─────────────┐  ┌─────────────────────┐ │
│  │ 第三方应用   │  │ 恶意N-API   │  │ 伪造IPC     │  │ 网络中间人         │ │
│  │ (JS/Native) │  │ 调用        │  │ 消息        │  │ 攻击者             │ │
│  └──────┬──────┘  └──────┬──────┘  └──────┬──────┘  └──────────┬──────────┘ │
└─────────┼────────────────┼────────────────┼─────────────────────┼──────────┘
          │                │                │                     │
          ▼                ▼                ▼                     ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                         信任边界 (Trust Boundary)                            │
├─────────────────────────────────────────────────────────────────────────────┤
│                          用户空间 (User Space)                               │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                         N-API 层                                     │    │
│  │  frameworks/js/napi/source/location_napi_*.cpp (17个文件)            │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                    │                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                      框架层 (Framework)                              │    │
│  │  frameworks/location_common/common/source/permission_manager.cpp     │    │
│  │  frameworks/native/locator_sdk/                                     │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
│                                    │                                         │
│  ┌─────────────────────────────────────────────────────────────────────┐    │
│  │                     SA 服务层 (Services)                             │    │
│  │  ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐ ┌─────────┐       │    │
│  │  │Locator  │ │  GNSS   │ │Network  │ │Passive  │ │Geocode  │       │    │
│  │  │SA:2802  │ │SA:2803  │ │SA:2804  │ │SA:2805  │ │SA:2801  │       │    │
│  │  └─────────┘ └─────────┘ └─────────┘ └─────────┘ └─────────┘       │    │
│  │              locationhub 进程                                       │    │
│  └─────────────────────────────────────────────────────────────────────┘    │
└─────────────────────────────────────────────────────────────────────────────┘
          │
          ▼
┌─────────────────────────────────────────────────────────────────────────────┐
│                          内核空间 (Kernel Space)                             │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────────────────┐   │
│  │ IPC / Binder    │  │ 文件系统        │  │ 硬件驱动 (HDI)              │   │
│  │                 │  │                 │  │ IGnssInterface/INetworkHDI  │   │
│  └─────────────────┘  └─────────────────┘  └─────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────────────────┘
```

### 1.2 攻击向量总结

| 攻击向量 | 入口点 | 风险等级 | 说明 |
|----------|--------|----------|------|
| **恶意应用** | N-API/NDK | 🔴 高 | 应用层直接调用API |
| **IPC伪造** | ILocatorService.idl | 🔴 高 | 进程间通信伪造 |
| **回调劫持** | ILocatorCallback | 🟡 中 | 回调对象被篡改 |
| **配置注入** | sa_profile/*.json | 🟢 低 | 配置文件篡改 |
| **网络中间人** | NetworkAbility | 🟡 中 | 网络定位数据篡改 |
| **HDI攻击** | HDI接口 | 🔴 高 | 硬件层攻击 |

---

## 2. 外部输入清单 (Attack Surface Inventory)

### 2.1 N-API 接口输入 (JavaScript层)

**模块注册点**:
- 文件: `frameworks/js/napi/source/location_napi_entry.cpp:379,384`
- 模块名: `geolocation`, `geoLocationManager`

| JS API | C++ 函数 | 文件 | 输入参数 | 风险点 |
|--------|----------|------|----------|--------|
| `getCurrentLocation` | `GetCurrentLocation` | location_napi_adapter.cpp | request对象 | 参数解析、权限 |
| `getLastLocation` | `GetLastLocation` | location_napi_adapter.cpp | 无 | 缓存读取 |
| `isLocationEnabled` | `IsLocationEnabled` | location_napi_system.cpp | 无 | - |
| `enableLocation` | `EnableLocation` | location_napi_system.cpp | 无 | 系统权限检查 |
| `disableLocation` | `DisableLocation` | location_napi_system.cpp | 无 | 系统权限检查 |
| `requestEnableLocation` | `RequestEnableLocation` | location_napi_system.cpp | 无 | UI调用 |
| `getAddressesFromLocation` | `GetAddressesFromLocation` | location_napi_adapter.cpp | 坐标 | 坐标范围校验 |
| `getAddressesFromLocationName` | `GetAddressesFromLocationName` | location_napi_adapter.cpp | 地址字符串 | 字符串长度 |
| `on`/`off` | `On`/`Off` | location_napi_event.cpp | event类型 | 枚举验证 |
| `addGnssGeofence` | `AddGnssGeofence` | geofence_napi.cpp | 围栏参数 | 半径/坐标范围 |
| `removeGnssGeofence` | `RemoveGnssGeofence` | geofence_napi.cpp | fenceId | ID有效性 |
| `addBeaconFence` | `AddBeaconFence` | beacon_fence_napi.cpp | 信标参数 | BLE数据验证 |
| `enableLocationMock` | `EnableLocationMock` | location_napi_adapter.cpp | 无 | 特权检查 |
| `setMockedLocations` | `SetMockedLocations` | location_napi_adapter.cpp | 位置数组 | 数组大小限制 |
| `getPoiInfo` | `GetPoiInfo` | location_napi_adapter.cpp | 查询参数 | 参数范围 |
| `getCountryCode` | `GetIsoCountryCode` | location_napi_system.cpp | 无 | - |
| `getLocatingRequiredData` | `GetLocatingRequiredData` | location_napi_adapter.cpp | 配置参数 | 参数验证 |

**证据**: 完整N-API清单见 `frameworks/js/napi/source/` 目录 (17个源文件)

### 2.2 C API (NDK) 输入

**头文件**: `interfaces/c_api/include/oh_location.h`

| API | 输入参数 | 风险点 |
|-----|----------|--------|
| `OH_Location_IsLocatingEnabled` | `bool* enabled` | 空指针检查 |
| `OH_Location_StartLocating` | `Location_RequestConfig*` | 配置参数全验证 |
| `OH_Location_StopLocating` | `Location_RequestConfig*` | 指针匹配验证 |

**配置参数结构** (`oh_location_type.h:176-233`):
```c
typedef struct Location_BasicInfo {
    double latitude;           // -90 到 90
    double longitude;          // -180 到 180
    double altitude;
    double accuracy;
    double speed;
    double direction;          // 0 到 360
    int64_t timeForFix;
    int64_t timeSinceBoot;
    double altitudeAccuracy;
    double speedAccuracy;
    double directionAccuracy;
    int64_t uncertaintyOfTimeSinceBoot;
    Location_SourceType locationSourceType;
} Location_BasicInfo;
```

**风险点**:
- 经纬度范围验证
- 精度值非负验证
- 方向值范围验证
- 时间戳有效性

### 2.3 IPC 接口输入

**IDL文件**: `frameworks/native/locator_sdk/ILocatorService.idl`

**共61个IPC方法**，按风险等级分类：

#### 🔴 高风险方法 (需要严格权限校验)

| Code | 方法名 | 输入参数 | 风险说明 |
|------|--------|----------|----------|
| 3 | StartLocating | RequestConfig + Callback | 权限绕过可导致未授权定位 |
| 4 | StopLocating | Callback对象 | 停止任意请求(需验证所有权) |
| 5 | GetCacheLocation | 无 | 敏感位置数据泄露 |
| 9 | EnableAbility | boolean | 开关控制(系统权限) |
| 30 | EnableLocationMock | 无 | 模拟位置(特权操作) |
| 31 | DisableLocationMock | 无 | 停止模拟 |
| 32 | SetMockedLocations | Location数组 | 注入虚假位置 |
| 45 | AddGnssGeofence | GeofenceConfig | 资源耗尽攻击 |
| 46 | RemoveGnssGeofence | fenceId | 删除他人围栏 |
| 57 | AddBeaconFence | BeaconConfig | BLE信标注入 |
| 58 | RemoveBeaconFence | fenceId | 删除他人围栏 |
| 60 | GetPoiInfo | 查询参数 | 信息泄露/资源耗尽 |

#### 🟡 中风险方法

| Code | 方法名 | 输入参数 | 风险说明 |
|------|--------|----------|----------|
| 1 | GetSwitchState | 无 | 状态查询 |
| 12 | GetAddressByCoordinate | 坐标 | 逆地理编码 |
| 13 | GetAddressByLocationName | 地址字符串 | 地理编码 |
| 27 | AddFence | Fence参数 | 地理围栏 |
| 28 | RemoveFence | fenceId | 围栏移除 |
| 42 | ReportLocation | Location + provider | 位置注入(需验证来源) |

#### 🟢 低风险方法

| Code | 方法名 | 说明 |
|------|--------|------|
| 16-19 | GnssStatus/Nmea回调注册 | 状态监听 |
| 22-23 | CachedLocation回调 | 缓存位置 |

**证据**: `frameworks/native/locator_sdk/ILocatorService.idl` 完整定义61个IPC方法

### 2.4 回调接口输入 (双向IPC)

**回调Proxy类** (8个):

| 回调类型 | Proxy文件 | 风险点 |
|----------|-----------|--------|
| ILocatorCallback | locator_callback_proxy.cpp | 位置报告被拦截 |
| IGnssStatusCallback | gnss_status_callback_proxy.cpp | GNSS状态伪造 |
| INmeaMessageCallback | nmea_message_callback_proxy.cpp | NMEA消息注入 |
| ICachedLocationsCallback | cached_locations_callback_proxy.cpp | 缓存位置泄露 |
| ICountryCodeCallback | country_code_callback_proxy.cpp | 国家码伪造 |
| ILocatingRequiredDataCallback | locating_required_data_callback_proxy.cpp | 数据泄露 |
| IBluetoothScanResultCallback | bluetooth_scan_result_callback.cpp | BLE扫描结果 |
| ILocationGnssGeofenceCallback | location_gnss_geofence_callback_proxy.cpp | 围栏事件伪造 |

**风险**: 回调对象被替换或劫持，导致数据泄露或伪造

### 2.5 配置文件输入

| 配置文件 | 路径 | 输入点 | 风险 |
|----------|------|--------|------|
| SA配置 | `sa_profile/*.json` | 启动参数 | 配置注入 |
| GNSS配置 | `services/location_gnss/gnss/etc/gnss_config.json` | 定位参数 | 参数篡改 |
| 日志配置 | `services/location_locator/hisysevent.yaml` | 事件配置 | 日志绕过 |

**证据**: `sa_profile/2801.json` - `sa_profile/2805.json`

### 2.6 网络输入

**NetworkAbility网络连接**:
- 文件: `services/location_network/network/source/network_ability.cpp:166`
- 连接云端网络定位服务
- **风险**: 中间人攻击、响应篡改

**AGnss网络接口**:
- 文件: `services/location_gnss/gnss/source/agnss_ni_manager.cpp:318`
- 辅助GNSS网络数据下载
- **风险**: 辅助数据篡改

**地理编码服务**:
- 文件: `services/location_geocode/geocode/source/geo_convert_service.cpp:158`
- 连接地理编码云端服务
- **风险**: 地址/坐标映射篡改

### 2.7 HDI硬件接口输入

**GNSS HDI接口**:
- 接口: `IGnssInterface`
- 文件: `services/location_gnss/gnss/source/gnss_ability.cpp` (HDI调用)
- **风险**: 硬件层位置伪造

**Geofence HDI接口**:
- 接口: `IGeofenceInterface`
- **风险**: 围栏事件伪造

---

## 3. 敏感操作清单 (Sensitive Operations)

### 3.1 权限检查点

| 检查点 | 文件路径 | 函数 | 权限要求 |
|--------|----------|------|----------|
| 定位权限 | `frameworks/location_common/common/source/permission_manager.cpp:29` | CheckLocationPermission | ohos.permission.LOCATION |
| 粗略定位 | `permission_manager.cpp:34-56` | CheckApproximatelyPermission | ohos.permission.APPROXIMATELY_LOCATION |
| 后台定位 | `permission_manager.cpp:67-81` | CheckBackgroundPermission | ohos.permission.LOCATION_IN_BACKGROUND |
| 模拟位置 | `permission_manager.cpp:84-94` | CheckMockLocationPermission | ohos.permission.MOCK_LOCATION |
| 系统权限 | `permission_manager.cpp:117-129` | CheckSystemPermission | 系统应用或特定Token |
| 开关控制 | `locator_background_proxy.cpp:195` | CheckPermission | CONTROL_LOCATION_SWITCH |

**关键权限常量** (`permission_manager.h`):
```cpp
"ohos.permission.LOCATION"                    // 精确定位
"ohos.permission.APPROXIMATELY_LOCATION"      // 粗略定位
"ohos.permission.LOCATION_IN_BACKGROUND"      // 后台定位
"ohos.permission.MOCK_LOCATION"               // 模拟位置
"ohos.permission.MANAGE_SECURE_SETTINGS"      // 安全设置
"ohos.permission.LOCATION_SWITCH_IGNORED"     // 忽略开关
"ohos.permission.CONTROL_LOCATION_SWITCH"     // 控制开关
```

### 3.2 系统调用/特权操作

| 操作 | 文件 | 函数 | 说明 |
|------|------|------|------|
| SA启动 | locator_ability.cpp:74 | MakeAndRegisterAbility | 注册系统服务 |
| UI对话框 | location_ui/ | UIExtensionAbility | 系统级UI |
| 网络连接 | network_ability.cpp | ConnectAbility | 连接云端 |
| HDI调用 | gnss_ability.cpp | HDI接口调用 | 硬件访问 |
| 回调注册 | locator_ability.cpp | RegisterXXXCallback | 跨进程回调 |

### 3.3 文件系统操作

| 操作 | 文件路径 | 说明 |
|------|----------|------|
| 配置读取 | `services/location_gnss/gnss/source/gnss_ability.cpp:1265,1286` | GNSS配置 |
| 配置管理 | `services/location_locator/locator/source/location_config_manager.cpp:71,87,146,272` | 定位配置 |
| 通用文件 | `frameworks/location_common/common/source/common_utils.cpp:621,637` | 文件工具 |

### 3.4 数据存储操作

| 操作 | 位置 | 说明 |
|------|------|------|
| 位置缓存 | report_manager.cpp | 缓存最近位置 |
| 请求记录 | request_manager.cpp | 存储活跃请求 |
| 围栏存储 | geofence模块 | 存储围栏配置 |

---

## 4. 信任边界跨越点 (Trust Boundary Crossings)

### 4.1 用户态→系统服务边界

**跨越点1: N-API入口**
```
JS App (不可信) 
    → N-API层 (frameworks/js/napi/)
    → PermissionManager::CheckXxxPermission() [权限检查点]
    → Inner API (frameworks/native/)
    → IPC调用
    → SA服务 (locationhub进程) [可信]
```

**跨越点2: IPC边界**
```
App进程 (不可信)
    → ILocatorService Proxy
    → IPC/Binder
    → LocatorServiceStub [信任边界]
    → LocatorAbility (验证调用者UID/PID)
```

### 4.2 系统服务→硬件边界

**跨越点3: HDI接口**
```
SA服务 (locationhub - 用户态可信)
    → HDI Proxy
    → IPC/Binder
    → HDI服务 (内核态驱动)
    → 硬件芯片
```

**跨越点4: 网络边界**
```
SA服务 (locationhub)
    → HTTP/TLS
    → 云端定位服务 [外部不可信网络]
    → 响应数据
    → [需验证响应完整性]
```

### 4.3 回调反向边界

**跨越点5: 回调通知**
```
SA服务 (可信)
    → ILocatorCallback Proxy
    → IPC/Binder
    → App进程 (不可信) [信任边界]
    → JS回调函数
```

---

## 5. 攻击面总览表

### 5.1 按组件分类

| 组件 | 输入点数量 | 敏感操作 | 风险等级 |
|------|-----------|----------|----------|
| N-API层 | 17+ | 参数解析、权限委托 | 🔴 高 |
| IPC接口 | 61 | 权限校验、请求路由 | 🔴 高 |
| Locator SA | 20+ | 请求管理、权限检查 | 🔴 高 |
| GNSS SA | 15+ | HDI调用、硬件访问 | 🔴 高 |
| Network SA | 10+ | 网络连接 | 🟡 中 |
| Geocode SA | 8+ | 网络服务 | 🟡 中 |
| Passive SA | 5+ | 被动监听 | 🟢 低 |

### 5.2 按攻击类型分类

| 攻击类型 | 相关输入点 | 风险等级 | 典型场景 |
|----------|-----------|----------|----------|
| **权限绕过** | N-API、IPC入口 | 🔴 高 | 未授权获取位置 |
| **输入验证绕过** | 所有参数输入点 | 🔴 高 | 越界、注入攻击 |
| **回调劫持** | 8个回调接口 | 🟡 中 | 拦截/伪造位置 |
| **资源耗尽** | IPC方法27,45,57 | 🟡 中 | DoS攻击 |
| **中间人攻击** | Network连接点 | 🟡 中 | 位置数据篡改 |
| **配置注入** | 配置文件 | 🟢 低 | 参数篡改 |

### 5.3 关键攻击路径

**路径1: 未授权定位**
```
恶意App → N-API StartLocating → IPC Code 3 
→ 权限检查绕过 → 成功获取位置
```

**路径2: 位置数据泄露**
```
恶意App → RegisterCallback → 等待其他App定位
→ ReportLocation(回调) → 获取他人位置
```

**路径3: 位置伪造**
```
攻击者 → EnableLocationMock → SetMockedLocations
→ 注入虚假位置 → 影响其他App
```

**路径4: 围栏绕过**
```
恶意App → RemoveGnssGeofence(fenceId)
→ 验证不严 → 删除他人围栏
```

---

## 6. 检查清单 (Security Review Checklist)

### 6.1 N-API层检查项

- [ ] 所有JS参数都经过类型检查
- [ ] 指针参数都经过空指针检查
- [ ] 枚举值都经过有效性验证
- [ ] 字符串长度都经过限制
- [ ] 数组大小都经过限制
- [ ] 回调函数都经过注册验证

### 6.2 IPC层检查项

- [ ] 所有IPC方法都检查调用者权限
- [ ] 敏感操作都验证Token ID
- [ ] 回调注册都验证应用身份
- [ ] 资源分配都有上限限制
- [ ] 跨应用操作都验证所有权

### 6.3 服务层检查项

- [ ] 请求对象都经过验证
- [ ] 位置报告都经过权限过滤
- [ ] 缓存数据都有访问控制
- [ ] 配置读取都经过验证
- [ ] 网络响应都经过校验

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [安全风险评审](07_SecurityReview.md) | 详细安全风险分析 |
| [系统架构](01_Architecture.md) | 架构与数据流 |
| [C/N-API接口](02_C_NAPI.md) | API安全考量 |
| [IPC接口](04_Inner_API.md) | 内部IPC接口 |

---

## 更新日志

| 日期 | 版本 | 变更 |
|------|------|------|
| 2026-02-07 | 1.0 | 初始版本，详细攻击面分析 |
