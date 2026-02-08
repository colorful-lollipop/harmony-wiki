# GN Targets 梳理

**目的**: 描述 WLAN 组件的 GN（Generate Ninja）构建系统，包括所有 targets、依赖关系、Feature Flags 和编译变体

**适用范围**: 构建工程师、组件开发者、系统集成者

**生成时间**: 2026-02-06

---

## 构建系统概览

WLAN 组件使用 GN 构建系统，支持标准系统和轻量级系统两种变体：

| 变体 | 定义 | 特点 |
|------|------|------|
| **Standard** | `!defined(ohos_lite)` | 完整功能，NAPI 模块，多个 SA，CFI/sanitizer |
| **Lite** | `defined(ohos_lite)` | 最小功能，单个服务库，无 NDK/NAPI |

---

## 构建组层次结构

### 层次关系
```
base_group (基础库)
    ↓
fwk_group (框架层)
    ├── wifi_sdk (Native SDK)
    ├── wifi_ndk (NDK)
    ├── wifi (NAPI 主模块)
    ├── wifiext (NAPI 扩展模块)
    ├── wifimanager (WiFi Manager NAPI)
    ├── wifimanagerext (WiFi Manager Ext NAPI)
    ├── cj_wifi_ffi (Cangjie FFI)
    └── wifiManager_framework_taihe (ArkTS WiFi Manager)
    ↓
service_group (服务组)
    ├── wifi_system_ability (SA 组)
    │   ├── wifi_device_ability (SA 1120)
    │   ├── wifi_scan_ability (SA 1124)
    │   ├── wifi_hotspot_ability (SA 1121)
    │   └── wifi_p2p_ability (SA 1123, 条件)
    └── wifi_manage (管理服务)
        ├── wifi_manager_service (核心服务)
        ├── wifi_ap_service (AP 服务)
        ├── wifi_p2p_service (P2P 服务)
        ├── wifi_scan_service (扫描服务)
        └── [其他子服务]
    ↓
relation_services (关联服务)
    └── portal_login_hap (应用)
```

---

## 关键 Targets 详解

### 1. Base Group Targets

#### wifi_ext_lib
**目标路径**: `//foundation/communication/wifi/wifi/utils/extern_library:wifi_ext_lib`

**属性**:
- **类型**: `ohos_shared_library`
- **输出**: `libwifi_ext_lib.so`
- **安装**: `install_enable = true`

**说明**: 基础工具库，提供国际化支持。

**Sources**:
- `wifi_intl_util.cpp`

**Dependencies**:
- `bounds_checking_function:libsec_shared`
- `c_utils:utils`
- `hilog:libhilog`
- `icu:shared_icuuc` (条件依赖）

**Defines**:
- `I18N_INTL_UTIL_ENABLE` (如果 global_i18n part 存在)

**安全配置**:
- 分支保护器: `pac_ret`
- CFI 启用: `-fsanitize=cfi`
- 跨 DSO CFI: `-fsanitize=cfi`

---

### 2. Framework Group Targets

#### wifi_sdk（Native SDK）
**目标路径**: `//foundation/communication/wifi/wifi/frameworks/native:wifi_sdk`

**属性**:
- **类型**: `ohos_shared_library`
- **输出**: `libwifi_sdk.so`
- **Inner API Tags**: `platformsdk`
- **版本脚本**: `libwifi_sdk.map`

**Sources (Standard)**: 30+ 源文件
- Proxy 实现：`wifi_device_proxy_impl`, `wifi_hotspot_proxy_impl`, `wifi_p2p_proxy_impl`, `wifi_scan_proxy_impl`
- Stub 实现：`wifi_device_callback_stub`, `wifi_hotspot_callback_stub`, `wifi_p2p_callback_stub`, `wifi_scan_callback_stub`
- 核心实现：`wifi_device.cpp`, `wifi_hotspot.cpp`, `wifi_p2p.cpp`, `wifi_scan.cpp`, `wifi_hid2d.cpp`, `wifi_msg.cpp`
- 事件处理：`wifi_sa_event.cpp`, `wifi_hid2d_msg.cpp`

**Internal Dependencies**:
- `:wifi_device_proxy_impl` (静态链接)
- `:wifi_hotspot_proxy_impl` (静态链接)
- `:wifi_p2p_proxy_impl` (静态链接)
- `:wifi_scan_proxy_impl` (静态链接)
- `$WIFI_ROOT_DIR/utils:wifi_utils`

**Defines**:
- `STA_INSTANCE_MAX_NUM=$wifi_feature_with_sta_num` (默认：2)
- `AP_INSTANCE_MAX_NUM=$wifi_feature_with_ap_num` (默认：1)
- `SUPPORT_RANDOM_MAC_ADDR` (条件定义)

**作用**: 提供 C++ Native SDK，封装所有 IPC Proxy 和 SA 接口。

---

#### wifi_ndk（NDK Library）
**目标路径**: `//foundation/communication/wifi/wifi/frameworks/wifi_ndk:wifi_ndk`

**属性**:
- **类型**: `ohos_shared_library` + `ohos_ndk_library`
- **输出**: `libwifi_ndk.so`
- **安装目录**: `ndk/`

**Sources**:
- `oh_wifi.cpp`

**NDK Headers**:
- `$WIFI_ROOT_DIR/interfaces/c_api/include/oh_wifi.h`

**Defines**:
- `API_EXPORT=__attribute__((visibility("default")))`

**Dependencies**:
- `$WIFI_ROOT_DIR/frameworks/native:wifi_sdk`

**作用**: 为第三方开发者提供 NDK 接口，可直接调用 WiFi SA。

---

#### wifi（主 NAPI 模块）
**目标路径**: `//foundation/communication/wifi/wifi/frameworks/js/napi:wifi`

**属性**:
- **类型**: `ohos_shared_library`
- **输出**: `wifi.z.so`

**Sources**:
- `wifi_napi_entry.cpp` - 主入口
- `wifi_napi_device.cpp` - STA 操作
- `wifi_napi_hotspot.cpp` - AP 操作
- `wifi_napi_p2p.cpp` - P2P 操作
- `wifi_napi_event.cpp` - 事件处理
- `wifi_napi_utils.cpp` - 工具函数
- `wifi_napi_errcode.cpp` - 错误码

**作用**: 主 N-API 模块，导出所有 WiFi STA/AP/P2P 功能。

---

#### wifiext（扩展 NAPI 模块）
**目标路径**: `//foundation/communication/wifi/wifi/frameworks/js/napi:wifiext`

**属性**:
- **类型**: `ohos_shared_library`
- **输出**: `wifiext.z.so`

**Sources**:
- `wifi_ext_napi_entry.cpp` - 扩展模块入口
- `wifi_ext_napi_hotspot.cpp` - 扩展热点功能

**条件**: `FEATURE_AP_EXTENSION` 宏定义时才编译

**作用**: 提供扩展功能（如功率模型管理）。

---

#### cj_wifi_ffi（Cangjie FFI）
**目标路径**: `//foundation/communication/wifi/wifi/frameworks/cj:cj_wifi_ffi`

**属性**:
- **类型**: `ohos_shared_library`
- **输出**: `libcj_wifi_ffi.so`

**条件**: `!ohos_lite`

**作用**: 为 Cangjie 语言提供 WiFi FFI 绑定。

---

#### wifiManager/wifiManager Framework）
**目标路径**: `//foundation/communication/wifi/wifi/frameworks/ets/taihe/wifiManager:wifiManager_framework_taihe`

**属性**:
- **类型**: `ohos_shared_library`
- **输出**: `wifimanager.z.so`

**条件**: `!ohos_lite`

**作用**: ArkTS WiFi Manager 框架实现。

---

### 3. Service Group Targets

#### wifi_system_ability（SA 组）
**目标路径**: `//foundation/communication/wifi/wifi/services/wifi_standard/wifi_framework:wifi_system_ability`

**属性**:
- **类型**: `group`

**Dependencies (Standard)**:
- `$WIFI_ROOT_DIR/services/wifi_standard/sa_profile:wifi_standard_sa_profile`
- `$WIFI_ROOT_DIR/services/wifi_standard/wifi_framework/wifi_manage:wifi_device_ability` (条件：!ohos_lite)
- `$WIFI_ROOT_DIR/services/wifi_standard/wifi_framework/wifi_manage:wifi_scan_ability` (条件：!ohos_lite)
- `$WIFI_ROOT_DIR/services/wifi_standard/wifi_framework/wifi_manage:wifi_hotspot_ability` (条件：wifi_feature_with_ap_num > 0)
- `$WIFI_ROOT_DIR/services/wifi_standard/wifi_framework/wifi_manage:wifi_p2p_ability` (条件：!ohos_lite && wifi_feature_with_p2p)

**作用**: 聚合所有 System Ability 目标。

---

#### wifi_device_ability（SA 1120）
**目标路径**: `//foundation/communication/wifi/wifi/services/wifi_standard/wifi_framework/wifi_manage/wifi_device_ability`

**属性**:
- **类型**: `ohos_shared_library`
- **输出**: `libwifi_device_ability.z.so`
- **SA ID**: 1120
- **SA Profile**: `1120.json`

**Sources**:
- `wifi_device_mgr_service_impl.cpp` - 服务实现
- `wifi_device_mgr_stub.h` - IPC Stub 定义

**关键代码**:
```cpp
// 注册点
const bool REGISTER_RESULT = SystemAbility::MakeAndRegisterAbility(
    WifiDeviceMgrServiceImpl::GetInstance().GetRefPtr());
```

**作用**: 提供 Station 设备管理服务（SA 1120）。

---

#### wifi_scan_ability（SA 1124）
**目标路径**: `//foundation/communication/wifi/wifi/services/wifi_standard/wifi_framework/wifi_manage/wifi_scan_ability`

**属性**:
- **类型**: `ohos_shared_library`
- **输出**: `libwifi_scan_ability.z.so`
- **SA ID**: 1124
- **SA Profile**: `1124.json`

**Sources**:
- `wifi_scan_mgr_service_impl.cpp` - 服务实现
- `wifi_scan_mgr_stub.h` - IPC Stub 定义

**作用**: 提供扫描管理服务（SA 1124）。

---

#### wifi_hotspot_ability（SA 1121）
**目标路径**: `//foundation/communication/wifi/wifi/services/wifi_standard/wifi_framework/wifi_manage/wifi_hotspot_ability`

**属性**:
- **类型**: `ohos_shared_library`
- **输出**: `libwifi_hotspot_ability.z.so`
- **SA ID**: 1121
- **SA Profile**: `1121.json`

**Sources**:
- `wifi_hotspot_mgr_service_impl.cpp` - 服务实现
- `wifi_hotspot_mgr_stub.h` - IPC Stub 定义

**作用**: 提供热点/AP 管理服务（SA 1121）。

---

#### wifi_p2p_ability（SA 1123）
**目标路径**: `//foundation/communication/wifi/wifi/services/wifi_standard/wifi_framework/wifi_manage/wifi_p2p_ability`

**属性**:
- **类型**: `ohos_shared_library`
- **输出**: `libwifi_p2p_ability.z.so`
- **SA ID**: 1123
- **SA Profile**: `1123.json`

**Sources**:
- `wifi_p2p_service_impl.cpp` - 服务实现
- `wifi_p2p_stub.h` - IPC Stub 定义

**条件**: `wifi_feature_with_p2p` 为 true 时编译

**作用**: 提供 P2P 管理服务（SA 1123）。

---

#### wifi_manage（管理服务组）
**目标路径**: `//foundation/communication/wifi/wifi/services/wifi_standard/wifi_framework:wifi_manage`

**属性**:
- **类型**: `group`

**Dependencies**:
- `$WIFI_ROOT_DIR/services/wifi_standard/wifi_framework/wifi_manage:wifi_manager_service` (始终)
- `$WIFI_ROOT_DIR/services/wifi_standard/wifi_framework/wifi_manage:wifi_ap:wifi_ap_service` (条件：!ohos_lite)
- `$WIFI_ROOT_DIR/services/wifi_standard/wifi_framework/wifi_manage:wifi_p2p:wifi_p2p_service` (条件：!ohos_lite && wifi_feature_with_p2p)

**作用**: 聚合所有管理服务实现。

---

#### wifi_manager_service（核心服务）
**目标路径**: `//foundation/communication/wifi/wifi/services/wifi_standard/wifi_framework/wifi_manage:wifi_manager_service`

**属性**:
- **类型**: `ohos_shared_library`
- **输出**: `libwifi_manager_service.so`
- **版本脚本**: `libwifi_manager.map`

**Internal Static Library**: `wifi_manager_service_static`

**Sources (Static - 关键模块)**:
- WiFi 控制器（`wifi_controller/`）：
  - `concrete_clientmode_manager.cpp`
  - `concrete_manager_state_machine.cpp`
  - `multi_sta_manager.cpp`
  - `multi_sta_state_machine.cpp`
  - `softap_manager.cpp`
  - `softap_manager_state_machine.cpp`
  - `wifi_service_scheduler.cpp`
  - `wifi_controller_state_machine.cpp`
- WiFi 子管理（`wifi_sub_manage/`）：
  - `wifi_common_service_manager.cpp`
  - `wifi_event_subscriber_manager.cpp`
  - `wifi_hotspot_manager.cpp`
  - `wifi_location_mode_observer.cpp`
  - `wifi_multi_vap_manager.cpp`
  - `wifi_p2p_manager.cpp`
  - `wifi_scan_manager.cpp`
  - `wifi_sta_manager.cpp`
  - `wifi_toggler_manager.cpp`
  - `wifi_manager.cpp`
  - `wifi_service_manager.cpp`
- WiFi 工具包（`wifi_toolkit/`）：
  - 配置管理：`config/`（`wifi_config_center.cpp`, `wifi_config_file_spec.cpp`, `wifi_scan_config.cpp`, `wifi_settings.cpp`）
  - 网络辅助：`net_helper/`（`arp_checker.cpp`, `dhcpd_interface.cpp`, `if_config.cpp`）
  - 工具函数：`utils/`（`wifi_encryption_util.cpp`, `wifi_randommac_helper.cpp`）
  - 资产管理：`wifi_asset/`（`wifi_asset_manager.cpp`）
- WiFi 网络选择：`network_select`（`network_selection.cpp`）
- WiFi 原生接口：`wifi_native/`（`client/hdi_client/`, `common/`）

**Service Dependencies**:
- `wifi_common:wifi_common_service`
- `wifi_native:wifi_native`
- `wifi_pro:wifi_pro`
- `wifi_scan:wifi_scan_service`
- `wifi_self_cure:wifi_self_cure`
- `wifi_sta:wifi_sta_service`
- `wifi_sta_ext:wifi_sta_ext_service`
- `wifi_toolkit:wifi_toolkit`
- `network_select:network_select`

**作用**: 核心服务实现，集成所有子服务和功能模块。

---

### 4. 子服务库 Targets

#### wifi_sta_service
**目标路径**: `//foundation/communication/wifi/wifi/services/wifi_standard/wifi_framework/wifi_manage/wifi_sta:wifi_sta_service`

**属性**:
- **类型**: `ohos_static_library` (standard) / `shared_library` (lite)

**Sources**:
- `sta_auto_connect_service.cpp`
- `sta_interface.cpp`
- `sta_monitor.cpp`
- `sta_saved_device_appraisal.cpp`
- `sta_service.cpp`
- `sta_state_machine.cpp`

**作用**: Station 模式服务实现。

---

#### wifi_scan_service
**目标路径**: `//foundation/communication/wifi/wifi/services/wifi_standard/wifi_framework/wifi_manage/wifi_scan:wifi_scan_service`

**属性**:
- **类型**: `ohos_static_library` (standard) / `shared_library` (lite)

**Sources**:
- `scan_interface.cpp`
- `scan_monitor.cpp`
- `scan_service.cpp`
- `scan_state_machine.cpp`

**作用**: 扫描服务实现。

---

#### wifi_ap_service
**目标路径**: `//foundation/communication/wifi/wifi/services/wifi_standard/wifi_framework/wifi_manage/wifi_ap:wifi_ap_service`

**属性**:
- **类型**: `ohos_shared_library`

**Sources**:
- `ap_idle_state.cpp`
- `ap_interface.cpp`
- `ap_monitor.cpp`
- `ap_root_state.cpp`
- `ap_service.cpp`
- `ap_started_state.cpp`
- `ap_state_machine.cpp`
- `ap_stations_manager.cpp`
- `wifi_ap_nat_manager.cpp`

**作用**: AP/热点服务实现。

---

#### wifi_p2p_service
**目标路径**: `//foundation/communication/wifi/wifi/services/wifi_standard/wifi_framework/wifi_manage/wifi_p2p:wifi_p2p_service`

**属性**:
- **类型**: `ohos_shared_library`

**Sources**:
- `p2p_state_machine.cpp`
- `p2p_interface.cpp`
- `p2p_monitor.cpp`
- `wifi_p2p_service.cpp`
- `wifi_p2p_group_manager.cpp`
- `wifi_p2p_device_manager.cpp`
- `hid2d/wifi_hid2d_service_utils.cpp`

**作用**: P2P 服务实现。

---

#### wifi_common_service
**目标路径**: `//foundation/communication/wifi/wifi/services/wifi_standard/wifi_framework/wifi_manage/wifi_common:wifi_common_service`

**属性**:
- **类型**: `ohos_static_library` (standard) / `shared_library` (lite)

**Sources**:
- `wifi_auth_center.cpp`
- `wifi_permission_helper.cpp`
- `wifi_internal_event_dispatcher.cpp`
- `wifi_protect_manager.cpp`
- `wifi_country_code/wifi_country_code_manager.cpp`
- `network_black_list/network_black_list_manager.cpp`
- `network_status_history/network_status_history_manager.cpp`

**作用**: 提供通用功能：权限验证、事件分发、保护管理、国家码管理等。

---

### 5. Relation Services Targets

#### relation_services
**目标路径**: `//foundation/communication/wifi/wifi/relation_services:relation_services`

**属性**:
- **类型**: `group`

**Dependencies**:
- `$WIFI_ROOT_DIR/relation_services/etc/init:etc` (始终)
- `$WIFI_ROOT_DIR/base/cRPC:crpc_server` (条件：!wifi_feature_with_hdi_wpa_supported)
- `//third_party/wpa_supplicant/wpa_supplicant-2.9:wpa_supplicant` (条件：ohos_lite)

**作用**: 关联服务组，包括 DHCP、配置和 cRPC 服务。

---

#### portal_login_hap
**目标路径**: `//foundation/communication/wifi/wifi/application/portal_login:portal_login_hap`

**属性**:
- **类型**: `ohos_app`
- **HAP 名称**: `PortalLogin`
- **安装目录**: `app/PortalLogin`

**Sub-targets**:
- `portal_login_js_assets` - ETS/JavaScript 资源
- `portal_login_resources` - 资源文件
- `portal_login_app_profile` - 应用配置

**配置**:
- `ets2abc = true` (启用 ETS 到 Ark 字节码)
- Entry module: `entry`
- 证书: `signature/portallogin.p7b`

**作用**: 门户认证登录应用。

---

## Feature Flags 完整参考

**证据**: `wifi/wifi.gni:20-82`

| 标志 | 默认值 | 说明 | 影响范围 |
|------|---------|------|----------|
| `wifi_feature_dynamic_unload_sa` | `false` | 动态卸载 SA | 所有 SA |
| `wifi_feature_with_p2p` | `true` | P2P 功能 | wifi_p2p_service, P2P API |
| `wifi_feature_with_rpt` | `true` | Repeater 功能 | wifi_ap_service |
| `wifi_feature_with_ap_intf` | `"wlan"` | AP 接口名称 | 网络接口 |
| `wifi_feature_with_ap_num` | `1` | 最大 AP 实例数 | wifi_ap_service |
| `wifi_feature_with_sta_num` | `2` | 最大 STA 实例数 | wifi_sta_service, wifi_sdk |
| `wifi_feature_with_auth_disable` | `false` | 禁用权限检查 | 所有服务 |
| `wifi_feature_with_dhcp_disable` | `false` | 禁用 DHCP | wifi_ap_service |
| `wifi_feature_with_encryption` | `true` | 配置加密 | wifi_toolkit |
| `wifi_feature_with_ap_extension` | `false` | AP 扩展 | wifi, wifiext |
| `wifi_feature_with_app_frozen` | `false` | 应用冻结检测 | wifi_manager_service |
| `wifi_feature_non_seperate_p2p` | `false` | 非分离 P2P | wifi_p2p_service |
| `wifi_feature_non_hdf_driver` | `false` | 非 HDF 驱动 | wifi_native |
| `wifi_feature_with_local_random_mac` | `true` | 本地随机 MAC | wifi_sta_service |
| `wifi_feature_wifi_pro_ctrl` | `true` | WiFi Pro 控制 | wifi_manager_service |
| `wifi_feature_voicewifi_enable` | `true` | 语音 WiFi | wifi_sta_service |
| `wifi_feature_with_data_report` | `false` | 数据报告 | wifi_sta_ext_service |
| `wifi_feature_sta_ap_exclusion` | `true` | STA-AP 互斥 | wifi_device_service_impl |
| `wifi_feature_with_random_mac_addr` | `true` | 随机 MAC 地址 | 多个服务 |
| `wifi_feature_with_hpf_supported` | `true` | HPF 支持 | wifi_sta_service |
| `wifi_feature_with_scan_control` | `true` | 扫描控制 | wifi_scan_service |
| `wifi_feature_with_hdi_wpa_supported` | `true` | HDI WPA 支持 | relation_services |
| `wifi_feature_network_selection` | `false` | 网络选择 | wifi_sta_service |
| `wifi_feature_p2p_random_mac_addr` | `true` | P2P 随机 MAC | wifi_p2p_service |
| `wifi_feature_powermgr_support` | `false` | 电源管理 | wifi_manager_service |
| `wifi_feature_with_sta_asset` | `true` | STA 资产 | wifi_toolkit |
| `wifi_feature_with_security_detect` | `true` | 安全检测 | wifi_manager_service |
| `wifi_feature_with_hdi_chip_supported` | `false` | HDI 芯片支持（条件） | wifi_toolkit |
| `wifi_feature_with_vap_manager` | `true` | VAP 管理器 | wifi_manager_service |
| `wifi_feature_mdm_restricted_enable` | `true` | MDM 限制 | wifi_device_service_impl |
| `wifi_feature_with_extensible_authentication` | `false` | 可扩展认证 | wifi_sta_service |
| `wifi_feature_with_scan_control_action_listen` | `true` | 扫描控制动作监听 | wifi_scan_service |
| `wifi_feature_autoopen_specified_location` | `true` | 自动打开指定位置 | 多个服务 |
| `wifi_feature_with_wifi_oeminfo_mac` | `false` | OEM 信息 MAC | wifi_sta_service |
| `wifi_feature_with_portal_login` | `true` | 门户登录 | wifi_sta_service |
| `wifi_feature_auto_enable_support` | `false` | 自动启用 WiFi | wifi_manager_service |
| `wifi_feature_with_ipv6_selfcure` | `true` | IPv6 自愈 | wifi_manager_service |
| `wifi_feature_with_dynamic_adjust_wifi_power_save` | `false` | 动态调整 WiFi 省电 | wifi_manager_service |
| `wifi_hiappevent_enable` | `false` | HiAppEvent | NAPI 模块 |
| `wifi_feature_with_local_security_detect` | `false` | 本地安全检测 | wifi_manager_service |

**条件依赖**:
- `wifi_feature_with_hdi_chip_supported`: 依赖 `global_parts_info.hdf_drivers_interface_wlan`
- `wifi_feature_with_app_frozen`: 依赖 `global_parts_info.resourceschedule_efficiency_manager`
- `wifi_feature_wifi_pro_ctrl`: 依赖 `global_parts_info.resourceschedule_ffrt`

---

## 编译配置

### 内存优化标志
**证据**: `wifi/wifi.gni:84-102`

```
memory_optimization_cflags = [
    "-fdata-sections",
    "-ffunction-sections",
]

memory_optimization_cflags_cc = [
    "-fvisibility-inlines-hidden",
    "-fdata-sections",
    "-ffunction-sections",
    "-fno-asynchronous-unwind-tables",
    "-fno-unwind-tables",
    "-fno-merge-all-constants",
    "-Os",
]

memory_optimization_ldflags = [
    "-Wl,--whole-archive",
    "-Wl,--gc-sections",
]
```

**用途**: 减小二进制体积，优化内存占用。

---

## 依赖树可视化

```
wifi.kits (Group)
├── wifi_sdk (ohos_shared_library)
│   ├── wifi_device_proxy_impl (静态)
│   ├── wifi_hotspot_proxy_impl (静态)
│   ├── wifi_p2p_proxy_impl (静态)
│   ├── wifi_scan_proxy_impl (静态)
│   ├── wifi_device_callback_stub (静态)
│   ├── wifi_hotspot_callback_stub (静态)
│   ├── wifi_p2p_callback_stub (静态)
│   ├── wifi_scan_callback_stub (静态)
│   └── 30+ other source files
│   └── utils:wifi_utils (依赖)
└── wifi (ohos_shared_library) [NAPI]
    └── 7 NAPI source files
```

---

## 构建产物映射

### Shared Libraries（标准系统）
| Target | 产物库 | 安装位置 | 用途 |
|--------|---------|----------|------|
| `wifi_ext_lib` | `libwifi_ext_lib.so` | `/system/lib/` | 基础工具 |
| `wifi_sdk` | `libwifi_sdk.so` | `/system/lib/` | Native SDK |
| `wifi_ndk` | `libwifi_ndk.so` | `/system/lib/` + `/ndk/` | NDK |
| `wifi_manager_service` | `libwifi_manager_service.so` | `/system/lib/` | 核心服务 |
| `wifi_device_ability` | `libwifi_device_ability.z.so` | `/system/lib/` | SA 1120 |
| `wifi_scan_ability` | `libwifi_scan_ability.z.so` | `/system/lib/` | SA 1124 |
| `wifi_hotspot_ability` | `libwifi_hotspot_ability.z.so` | `/system/lib/` | SA 1121 |
| `wifi_p2p_ability` | `libwifi_p2p_ability.z.so` | `/system/lib/` | SA 1123 |
| `wifi` | `wifi.z.so` | `/system/lib/module/` | NAPI 主模块 |
| `wifiext` | `wifiext.z.so` | `/system/lib/module/` | NAPI 扩展模块 |
| `wifimanager` | `wifimanager.z.so` | `/system/lib/module/` | WiFi Manager |
| `wifimanagerext` | `wifimanagerext.z.so` | `/system/lib/module/` | WiFi Manager Ext |
| `cj_wifi_ffi` | `libcj_wifi_ffi.so` | `/system/lib/` | Cangjie FFI |

### Static Libraries（链接到共享库）
| 静态库 | 被链接到 | 用途 |
|---------|-----------|------|
| `wifi_toolkit.a` | wifi_manager_service | 工具包 |
| `wifi_sta_service.a` | wifi_manager_service | STA 服务 |
| `wifi_scan_service.a` | wifi_manager_service | 扫描服务 |
| `wifi_common_service.a` | wifi_manager_service | 通用功能 |
| `wifi_manager_service_static.a` | wifi_manager_service | 静态库 |
| `wifi_native.a` | wifi_manager_service | 原生接口 |
| `wifi_pro.a` | wifi_manager_service | WiFi Pro |
| `wifi_scan.a` | wifi_manager_service | 扫描原生 |
| `wifi_self_cure.a` | wifi_manager_service | 自愈 |
| `network_select.a` | wifi_manager_service | 网络选择 |

### 应用
| Target | 产物 | 安装位置 |
|--------|---------|----------|
| `portal_login_hap` | `PortalLogin.hap` | `/system/app/PortalLogin/` |

---

## 构建命令示例

### 构建标准系统
```bash
# 配置 feature flags
./build.sh --product-name=OHOS \
  --build-target=standard_system \
  --ccache \
  --gn-args="wifi_feature_with_p2p=true wifi_feature_with_scan_control=true"
```

### 构建轻量级系统
```bash
./build.sh --product-name=OHOS \
  --build-target=small_system \
  --gn-args="ohos_lite=true wifi_feature_with_p2p=false"
```

### 添加 feature
```bash
./build.sh --gn-args="wifi_feature_wifi_pro_ctrl=true"
```

---

## 相关跳转

- [00_Overview.md](00_Overview.md) - 项目概览
- [01_Directory_Structure.md](01_Directory_Structure.md) - 目录结构
- [02_Architecture.md](02_Architecture.md) - 架构说明
- [06_Build_Artifacts.md](06_Build_Artifacts.md) - 编译产物详解
- [appendix/Config_Flags.md](appendix/Config_Flags.md) - Feature Flags 详解

---

**下一步**: 阅读 [06_Build_Artifacts.md](06_Build_Artifacts.md) 了解编译产物的安装和运行时加载关系。
