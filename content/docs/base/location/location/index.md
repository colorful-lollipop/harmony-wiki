# OpenHarmony 位置服务概览

> **目的**: 帮助新人快速理解 OpenHarmony 位置服务组件的定位、边界和核心能力  
> **适用范围**: 所有需要了解或使用位置服务的开发者  
> **最后更新**: 2026-02-07

---

## 项目定位

OpenHarmony 位置服务（Location Service）是 OpenHarmony 系统的基础能力组件，为应用提供设备位置定位能力。

### 核心职责

1. **位置获取**: 提供实时准确的位置数据
2. **多源融合**: 支持 GNSS、网络定位、基站定位、WLAN/蓝牙定位
3. **权限管理**: 保护用户位置隐私，管理应用定位权限
4. **地理编码**: 提供坐标与地址的相互转换能力

### 在系统中的位置

```
用户应用
    ↓
JS/C API 接口层 (frameworks/js/napi, interfaces/c_api)
    ↓
框架层 (frameworks/location_common, frameworks/native)
    ↓
SA 服务层 (services/location_*)
    ↓
HAL/HDI 层 (硬件抽象)
```

---

## 核心能力

### 1. 定位技术

| 技术 | 说明 | 精度 | 功耗 |
|------|------|------|------|
| GNSS | 卫星定位（GPS/GLONASS/北斗等） | 高 | 高 |
| 网络定位 | 基站/WLAN/蓝牙定位 | 中低 | 低 |
| 室内定位 | 高精度室内定位 | 高 | 中 |
| RTK | 实时动态定位 | 极高 | 高 |

### 2. 使用场景

- **导航**: 车载导航、行人导航
- **运动记录**: 跑步、骑行轨迹记录
- **出行**: 打车、公共交通
- **生活服务**: 新闻、购物、外卖（粗略位置）

### 3. 地理编码

- 坐标 → 地址（逆地理编码）
- 地址 → 坐标（地理编码）
- 周边搜索（POI）

---

## 关键概念

### 坐标系

系统使用 **WGS84** 坐标系描述地球上的位置：

- **纬度 (Latitude)**: -90 到 90，正值为北纬
- **经度 (Longitude)**: -180 到 180，正值为东经

### 位置开关

用户必须开启位置开关，系统才会提供定位服务。应用可通过 API 查询开关状态。

### 权限体系

位置信息属于用户敏感数据，需要用户授权：

| 权限 | 说明 | 精度 |
|------|------|------|
| `ohos.permission.APPROXIMATELY_LOCATION` | 粗略位置 | 城市级 |
| `ohos.permission.LOCATION` | 精确位置 | 设备级 |

---

## 系统能力 (SysCap)

位置服务组件声明以下系统能力：

| SysCap | 说明 |
|--------|------|
| `SystemCapability.Location.Location.Core` | 核心定位能力 |
| `SystemCapability.Location.Location.Gnss` | GNSS 定位能力 |
| `SystemCapability.Location.Location.Geofence` | 地理围栏能力 |
| `SystemCapability.Location.Location.Geocoder` | 地理编码能力 |
| `SystemCapability.Location.Location.Lite` | 轻量级定位 |

> 证据: `bundle.json:18-24`

---

## 目录结构

```
base/location/location/
├── figures/                 # 架构图
├── frameworks/              # 框架层代码
│   ├── location_common/     # 公共框架
│   ├── native/              # Native 框架
│   ├── js/                  # JS N-API 框架
│   ├── ets/                 # ETS 框架
│   └── cj/                  # CJ 框架
├── interfaces/              # 对外接口
│   ├── inner_api/           # 内部 API
│   └── c_api/               # C API
├── sa_profile/              # SA 配置文件
├── services/                # SA 服务代码
│   ├── location_locator/    # 主定位器 SA
│   ├── location_gnss/       # GNSS 定位 SA
│   ├── location_network/    # 网络定位 SA
│   ├── location_passive/    # 被动定位 SA
│   ├── location_geocode/    # 地理编码 SA
│   └── location_ui/         # 定位对话框 SA
└── test/                    # 测试代码
```

---

## SA 服务列表

| SA ID | 服务名 | 职责 | 产物 |
|-------|--------|------|------|
| SA ID | 服务名 | 职责 | 产物 |
|-------|--------|------|------|
| 2801 | Geocode | 地理编码服务 | `liblbsservice_geocode.z.so` |
| 2802 | Locator | 主定位服务 | `liblbsservice_locator.z.so` |
| 2803 | GNSS | GNSS 定位服务 | `liblbsservice_gnss.z.so` |
| 2804 | Network | 网络定位服务 | - (内嵌) |
| 2805 | Passive | 被动定位服务 | - (内嵌) |

> 证据: `sa_profile/*.json`

---

## 编译产物

主要产物：

| 产物 | 类型 | 说明 |
|------|------|------|
| `libohlocation.so` | 动态库 | C API 库 |
| `liblocator_sdk.so` | 动态库 | Native SDK |
| `libgeolocation.z.so` | 动态库 | JS N-API |
| `lbsservice_*.so` | 动态库 | 各 SA 服务 |

> 详细说明: [编译产物](06_Artifacts.md)

---

## 依赖关系

### 系统依赖

- **IPC**: IPC 框架（进程间通信）
- **SAMgr**: 系统能力管理
- **Privacy**: 隐私权限管理
- **HDF**: 硬件驱动框架
- **AbilityRuntime**: 能力运行时

### 组件依赖

```
location
├── ability_base
├── ability_runtime
├── access_token
├── ipc
├── safwk
├── samgr
├── hiview
└── ... (更多依赖)
```

> 证据: `bundle.json:43-93`

---

## Feature 开关

可通过 `config.gni` 配置的 Feature：

| Feature | 默认值 | 说明 |
|---------|--------|------|
| `location_feature_with_geocode` | true | 地理编码功能 |
| `location_feature_with_gnss` | true | GNSS 定位功能 |
| `location_feature_with_network` | true | 网络定位功能 |
| `location_feature_with_passive` | true | 被动定位功能 |
| `location_feature_with_jsstack` | true | JS 栈支持 |
| `location_sa_recycle_strategy_low_memory` | true | 低内存回收策略 |
| `location_feature_with_opp_switch` | true | OPP 开关支持 |
| `location_feature_with_quick_unload` | true | 快速卸载 |

> 证据: `config.gni:38-69`

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [系统架构](01_Architecture.md) | 详细架构说明 |
| [C/N-API 接口](02_C_NAPI.md) | C API 参考 |
| [JS API 接口](03_JS_API.md) | JS API 参考 |
| [GN 构建配置](05_Build.md) | 构建配置说明 |
| [安全风险评审](07_Security.md) | 安全分析 |

---

## 快速开始

### 1. 引入依赖

在应用中声明使用位置能力：

```json
"requestPermissions": [
  {
    "name": "ohos.permission.LOCATION",
    "usedScene": {
      "when": "inuse"
    }
  }
]
```

### 2. 调用 API

**JS 调用示例**:
```javascript
import geolocation from '@ohos.geolocation';
```

**C 调用示例**:
```c
#include "oh_location.h"
```

### 3. 处理结果

位置信息通过回调返回，包含经纬度、精度、时间戳等。

---

## 更新日志

| 日期 | 版本 | 变更 |
|------|------|------|
| 2026-02-05 | 1.0 | 初始版本 |
