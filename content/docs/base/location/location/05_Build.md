# GN 构建配置

> **目的**: 详细说明位置服务组件的 GN 构建配置、targets 依赖关系和构建产物  
> **适用范围**: 构建工程师、集成工程师、需要理解构建系统的开发者  
> **最后更新**: 2026-02-05

---

## 1. 配置文件概览

### 1.1 关键配置文件

| 文件 | 作用 |
|------|------|
| `config.gni` | 全局配置，定义 feature 开关和编译参数 |
| `bundle.json` | 组件配置，定义组件元数据、依赖和构建目标 |
| `sa_profile/BUILD.gn` | SA 配置文件构建 |
| 各模块 `BUILD.gn` | 模块级构建配置 |

> 证据: `bundle.json`, `config.gni`, `sa_profile/BUILD.gn`

### 1.2 配置结构

```
base/location/location/
├── config.gni                    # 全局配置
├── bundle.json                   # 组件配置
├── sa_profile/
│   ├── BUILD.gn                  # SA Profile 构建
│   ├── 2801.json ~ 2805.json     # SA 定义
│   └── *_dynamic_offload.json    # 动态卸载配置
├── frameworks/
│   ├── js/napi/BUILD.gn          # JS N-API 构建
│   ├── native/
│   │   ├── locator_sdk/BUILD.gn
│   │   └── location_ndk/BUILD.gn
│   └── location_common/BUILD.gn
└── services/
    ├── location_locator/BUILD.gn
    ├── location_gnss/BUILD.gn
    └── ...
```

---

## 2. 全局配置 (config.gni)

### 2.1 Feature 开关

```gni
// 文件: config.gni:38-69

# 核心功能开关
location_feature_with_geocode = true      # 地理编码功能
location_feature_with_gnss = true         # GNSS 定位功能
location_feature_with_network = true      # 网络定位功能
location_feature_with_passive = true      # 被动定位功能
location_feature_with_jsstack = true      # JS 栈支持

# 策略开关
location_sa_recycle_strategy_low_memory = true   # 低内存回收策略
location_feature_with_opp_switch = true          # OPP 开关支持
location_feature_with_quick_unload = true        # 快速卸载

# 依赖组件开关
i18n_enable = true
telephony_core_service_enable = true
telephony_cellular_data_enable = true
hdf_drivers_interface_location_gnss_enable = true
hdf_drivers_interface_location_agnss_enable = true
communication_wifi_enable = true
communication_bluetooth_enable = true
```

### 2.2 目录变量

```gni
// 文件: config.gni:14-36

LOCATION_ROOT_DIR = "//base/location/location"
SUBSYSTEM_DIR = "$LOCATION_ROOT_DIR/services"
LOCATION_GNSS_ROOT = "$SUBSYSTEM_DIR/location_gnss/gnss"
LOCATION_LOCATOR_ROOT = "$SUBSYSTEM_DIR/location_locator/locator"
LOCATION_GEOCONVERT_ROOT = "$SUBSYSTEM_DIR/location_geocode/geocode"
LOCATION_NETWORK_ROOT = "$SUBSYSTEM_DIR/location_network/network"
LOCATION_PASSIVE_ROOT = "$SUBSYSTEM_DIR/location_passive/passive"
LOCATION_NATIVE_DIR = "$LOCATION_ROOT_DIR/frameworks/native"
LOCATION_COMMON_DIR = "$LOCATION_ROOT_DIR/frameworks/location_common/common"
```

---

## 3. 组件配置 (bundle.json)

### 3.1 组件元数据

```json
{
  "name": "@ohos/location",
  "version": "3.1.0",
  "subsystem": "location",
  "syscap": [
    "SystemCapability.Location.Location.Core",
    "SystemCapability.Location.Location.Gnss",
    "SystemCapability.Location.Location.Geofence",
    "SystemCapability.Location.Location.Geocoder",
    "SystemCapability.Location.Location.Lite"
  ]
}
```

### 3.2 构建目标分组

```json
// 文件: bundle.json:94-124

"build": {
  "group_type": {
    "base_group": [
      "//base/location/location/services/utils:lbsresources",
      "//base/location/location/frameworks/base_module:lbsbase_module"
    ],
    "fwk_group": [
      "//base/location/location/frameworks/native/locator_sdk:locator_sdk",
      "//base/location/location/frameworks/js/napi:geolocation",
      "//base/location/location/frameworks/js/napi:geolocationmanager",
      "//base/location/location/frameworks/native/locator_agent:locator_agent",
      "//base/location/location/frameworks/native/geofence_sdk:geofence_sdk",
      "//base/location/location/frameworks/native/location_ndk:location_ndk"
    ],
    "service_group": [
      "//base/location/location/services/location_geocode/geocode:lbsservice_geocode",
      "//base/location/location/services/location_gnss/gnss:lbsservice_gnss",
      "//base/location/location/services/location_locator/locator:lbsservice_locator",
      "//base/location/location/services/location_network/network:lbsservice_network",
      "//base/location/location/services/location_passive/passive:lbsservice_passive",
      "//base/location/location/services/location_ui:location_dialog_hap"
    ]
  }
}
```

### 3.3 内部 kits

```json
// 文件: bundle.json:125-166

"inner_kits": [
  {
    "header": {
      "header_base": "//base/location/location/interfaces/inner_api/include",
      "header_files": ["locator_impl.h"]
    },
    "name": "//base/location/location/frameworks/native/locator_sdk:locator_sdk"
  },
  {
    "name": "//base/location/location/frameworks/location_common/common:lbsservice_common"
  },
  {
    "name": "//base/location/location/frameworks/native/locator_agent:locator_agent"
  }
]
```

---

## 4. 关键 Targets

### 4.1 服务层 Targets

| Target | 类型 | 输出 | 依赖 |
|--------|------|------|------|
| `lbsservice_geocode` | shared_library | `liblbsservice_geocode.z.so` | geocode 模块 |
| `lbsservice_gnss` | shared_library | `liblbsservice_gnss.z.so` | gnss 模块 |
| `lbsservice_locator` | shared_library | `liblbsservice_locator.z.so` | locator 模块 |
| `lbsservice_network` | shared_library | `liblbsservice_network.z.so` | network 模块 |
| `lbsservice_passive` | shared_library | `liblbsservice_passive.z.so` | passive 模块 |

### 4.2 框架层 Targets

| Target | 类型 | 输出 | 依赖 |
|--------|------|------|------|
| `locator_sdk` | shared_library | `liblocator_sdk.so` | locator_agent |
| `geolocation` | shared_library | `libgeolocation.z.so` | js/napi |
| `location_ndk` | shared_library | `libohlocation.so` | c_api |
| `locator_agent` | shared_library | `liblocator_agent.z.so` | location_common |
| `geofence_sdk` | shared_library | `libgeofence_sdk.z.so` | location_common |

---

## 5. SA Profile 构建

### 5.1 BUILD.gn 配置

```gni
// 文件: sa_profile/BUILD.gn:17-37

ohos_sa_profile("location_sa_profile") {
  if (location_sa_recycle_strategy_low_memory) {
    sources = [
      "2801.json",
      "2802.json",
      "2803.json",
      "2804.json",
      "2805.json",
    ]
  } else {
    sources = [
      "2801_dynamic_offload.json",
      "2802_dynamic_offload.json",
      "2803_dynamic_offload.json",
      "2804_dynamic_offload.json",
      "2805_dynamic_offload.json",
    ]
  }

  subsystem_name = "location"
}
```

### 5.2 SA 配置文件示例

```json
// 文件: sa_profile/2801.json
{
  "process": "locationhub",
  "systemability": [
    {
      "name": 2801,
      "libpath": "liblbsservice_geocode.z.so",
      "run-on-create": false,
      "distributed": false,
      "dump_level": 1,
      "recycle-strategy": "low-memory"
    }
  ]
}
```

---

## 6. 条件编译

### 6.1 Feature 条件编译

```cpp
// 文件: services/location_locator/locator/source/locator_ability.cpp:31-50

#ifdef FEATURE_GEOCODE_SUPPORT
#include "geo_convert_proxy.h"
#endif

#ifdef FEATURE_GNSS_SUPPORT
#include "gnss_ability_proxy.h"
#endif

#ifdef FEATURE_NETWORK_SUPPORT
#include "network_ability_proxy.h"
#endif

#ifdef FEATURE_PASSIVE_SUPPORT
#include "passive_ability_proxy.h"
#endif
```

---

## 7. 编译产物映射

### 7.1 产物清单

| 产物路径 | 类型 | 说明 |
|----------|------|------|
| `system/lib64/libohlocation.z.so` | NDK 库 | C API |
| `system/lib64/liblocator_sdk.z.so` | SDK 库 | Native SDK |
| `system/lib64/module/libgeolocation.z.so` | N-API 库 | JS 接口 |
| `system/lib64/module/libgeolocationmanager.z.so` | N-API 库 | LocationManager |
| `system/lib64/sa_dynamic_libs/liblbsservice_geocode.z.so` | SA 库 | Geocode 服务 |
| `system/lib64/sa_dynamic_libs/liblbsservice_locator.z.so` | SA 库 | Locator 服务 |
| `system/lib64/sa_dynamic_libs/liblbsservice_gnss.z.so` | SA 库 | GNSS 服务 |
| `system/lib64/sa_dynamic_libs/liblbsservice_network.z.so` | SA 库 | Network 服务 |
| `system/lib64/sa_dynamic_libs/liblbsservice_passive.z.so` | SA 库 | Passive 服务 |
| `system/app/location_dialog/location_dialog.hap` | HAP | 定位权限对话框 |

---

## 相关文档

| 文档 | 说明 |
|------|------|
| [概览](index.md) | 项目定位与核心能力 |
| [编译产物](06_Artifacts.md) | 详细产物说明 |
| [系统架构](01_Architecture.md) | 架构与组件关系 |

---

## 更新日志

| 日期 | 版本 | 变更 |
|------|------|------|
| 2026-02-05 | 1.0 | 初始版本 |
