# GN Targets 与编译产物

> **目的**: 详细记录 battery_manager 项目的 GN 构建目标、依赖关系、输出产物和安装路径

**适用范围**: GN 构建系统、编译产物、安装配置

---

## 核心构建 Targets

### Framework 层 Targets

#### battery_napi (group)

**文件路径**: `frameworks/BUILD.gn:16-22`

**类型**: group

**依赖**:
- `napi:battery`
- `napi:batteryinfo`
- `napi:charger`

**说明**: 聚合所有 N-API 模块

**证据**: `frameworks/BUILD.gn:16-22`

#### batteryinfo (ohos_shared_library)

**文件路径**: `frameworks/napi/BUILD.gn:20-42`

**类型**: ohos_shared_library

**sources**:
- `src/battery_info.cpp`
- `src/napi_error.cpp`
- `src/napi_utils.cpp`

**deps**:
- `${battery_inner_api}:batterysrv_client`

**external_deps**:
- `c_utils:utils`
- `hilog:libhilog`
- `ipc:ipc_core`
- `napi:ace_napi`

**configs**:
- `${battery_utils}:utils_config`
- `:batterynapi_private_config`

**relative_install_dir**: module

**输出**: libbatteryinfo.z.so

**证据**: `frameworks/napi/BUILD.gn:20-42`

#### battery (ohos_shared_library)

**文件路径**: `frameworks/napi/BUILD.gn:63-82`

**类型**: ohos_shared_library

**sources**:
- `src/system_battery.cpp`

**deps**:
- `${battery_inner_api}:batterysrv_client`

**external_deps**:
- `c_utils:utils`
- `hilog:libhilog`
- `ipc:ipc_core`
- `napi:ace_napi`

**configs**:
- `${battery_utils}:utils_config`
- `${battery_utils}:coverage_flags`
- `:batterynapi_private_config`

**relative_install_dir**: module

**输出**: libbattery.z.so

**证据**: `frameworks/napi/BUILD.gn:63-82`

#### charger (ohos_shared_library)

**文件路径**: `frameworks/napi/BUILD.gn:44-61`

**类型**: ohos_shared_library

**sources**:
- `src/charger.cpp`

**deps**:
- `${battery_inner_api}:batterysrv_client`

**external_deps**:
- `c_utils:utils`
- `hilog:libhilog`
- `ipc:ipc_core`
- `napi:ace_napi`

**configs**:
- `${battery_utils}:utils_config`
- `${battery_utils}:coverage_flags`

**relative_install_dir**: module

**输出**: libcharger.z.so

**证据**: `frameworks/napi/BUILD.gn:44-61`

---

### Service 层 Targets

#### batteryservice (ohos_shared_library)

**文件路径**: `services/BUILD.gn:49-149`

**类型**: ohos_shared_library

**sources**:
- `utils/native/src/battery_xcollie.cpp`
- `native/src/battery_callback.cpp`
- `native/src/battery_config.cpp`
- `native/src/battery_dump.cpp`
- `native/src/battery_light.cpp`
- `native/src/battery_notify.cpp`
- `native/src/battery_service.cpp`

**configs**:
- `${battery_utils}:utils_config`
- `${battery_utils}:coverage_flags`
- `:batterysrv_public_config`

**public_configs**:
- `${battery_service_zidl}:batterysrv_public_config`
- `:batterysrv_public_config`

**deps**:
- `${battery_service_zidl}:batterysrv_stub`
- `${battery_utils}/hookmgr:battery_hookmgr`

**external_deps**:
- `ability_base:want`
- `ability_runtime:ability_manager`
- `bundle_framework:appexecfwk_base`
- `cJSON:cjson`
- `c_utils:utils`
- `common_event_service:cesfwk_core`
- `common_event_service:cesfwk_innerkits`
- `drivers_interface_battery:libbattery_proxy_2.0`
- `eventhandler:libeventhandler`
- `ffrt:libffrt`
- `hdf_core:libhdi`
- `hdf_core:libpub_utils`
- `hicollie:libhicollie`
- `hilog:libhilog`
- `ipc:ipc_core`
- `init:libbegetutil`
- `power_manager:power_ffrt`
- `power_manager:power_sysparam`
- `power_manager:power_vibrator`
- `power_manager:powermgr_client`
- `safwk:system_ability_fwk`
- `samgr:samgr_proxy`
- `power_manager:power_permission`

**条件 defines**:
- `CONFIG_USE_JEMALLOC_DFX_INTF` (use_musl && musl_use_jemalloc && musl_use_jemalloc_dfx_intf)
- `BATTERY_MANAGER_ENABLE_CHARGING_SOUND` (battery_manager_feature_enable_charging_sound)
- `BATTERY_MANAGER_ENABLE_WIRELESS_CHARGE` (battery_manager_feature_enable_wireless_charge)
- `BATTERY_MANAGER_SET_LOW_CAPACITY_THRESHOLD` (battery_manager_feature_set_low_capacity_threshold)
- `BATTERY_SUPPORT_NOTIFICATION` (battery_manager_feature_support_notification)
- `HAS_BATTERY_CONFIG_POLICY_PART` (has_battery_config_policy_part)
- `BATTERY_USER_VERSION` (build_variant == "user")
- `HAS_SENSORS_MISCDEVICE_PART` (has_sensors_miscdevice_part)

**输出**: libbatteryservice.z.so

**证据**: `services/BUILD.gn:49-149`

#### batterysrv_client (ohos_shared_library)

**文件路径**: `interfaces/inner_api/BUILD.gn:20-56`

**类型**: ohos_shared_library

**sources**:
- `${battery_frameworks}/native/src/battery_srv_client.cpp`

**deps**:
- `${battery_service_zidl}:batterysrv_proxy`

**configs**:
- `${battery_utils}:coverage_flags`

**public_configs**:
- `${battery_service_zidl}:batterysrv_public_config`
- `${battery_utils}:utils_config`
- `:batterysrv_public_config`

**external_deps**:
- `c_utils:utils`
- `hilog:libhilog`
- `ipc:ipc_core`
- `samgr:samgr_proxy`

**install_images**: system_base_dir

**relative_install_dir**: platformsdk

**输出**: libbatterysrv_client.z.so

**innerapi_tags**: platformsdk, sasdk

**证据**: `interfaces/inner_api/BUILD.gn:20-56`

#### battery_notification (ohos_shared_library)

**文件路径**: `services/BUILD.gn:151-193`

**类型**: ohos_shared_library

**sources**:
- `native/notification/button_event.cpp`
- `native/notification/notification_center.cpp`
- `native/notification/notification_decorator.cpp`
- `native/notification/notification_locale.cpp`
- `native/notification/notification_manager.cpp`

**include_dirs**: `native/notification`

**configs**:
- `${battery_utils}:utils_config`
- `${battery_utils}:coverage_flags`
- `:batterysrv_public_config`

**public_configs**:
- `:batterysrv_public_config`

**deps**:
- `${battery_inner_api}:batterysrv_client`

**external_deps**:
- `ability_base:want`
- `ability_base:zuri`
- `cJSON:cjson`
- `c_utils:utils`
- `config_policy:configpolicy_util`
- `distributed_notification_service:ans_innerkits`
- `hilog:libhilog`
- `i18n:intl_util`
- `image_framework:image_native`
- `ipc:ipc_single`

**输出**: libbattery_notification.z.so

**证据**: `services/BUILD.gn:151-193`

#### service (group)

**文件路径**: `services/BUILD.gn:195-221`

**类型**: group

**deps**:
- `:battery_notification`
- `:batteryservice`
- `native/profile:battery_config`
- `native/profile:battery_vibrator_config`

**条件 deps**:
- `:charging_sound` (battery_manager_feature_enable_charging_sound)
- `native/resources:battery_locale_path` (battery_manager_feature_support_notification)
- 多语言资源 (battery_manager_feature_support_notification_string)

**说明**: 聚合所有服务层 target

**证据**: `services/BUILD.gn:195-221`

---

### ZIDL 层 Targets

#### batterysrv_interface (idl_gen_interface)

**文件路径**: `services/zidl/BUILD.gn:24-31`

**类型**: idl_gen_interface

**sources**: `IBatterySrv.idl`

**log_domainid**: 0xD002922

**log_tag**: BatterySvc

**subsystem_name**: powermgr

**part_name**: battery_manager

**说明**: 生成 BatteryService 的 IPC Skeleton/Proxy

**证据**: `services/zidl/BUILD.gn:24-31`

#### batterysrv_proxy (ohos_source_set)

**文件路径**: `services/zidl/BUILD.gn:33-55`

**类型**: ohos_source_set

**sources**: `*_proxy.cpp` (生成的 Proxy 代码)

**deps**: `:batterysrv_interface`

**configs**:
- `${battery_utils}:utils_config`
- `:batterysrv_public_config`

**external_deps**:
- `c_utils:utils`
- `hilog:libhilog`
- `ipc:ipc_single`
- `samgr:samgr_proxy`

**说明**: IPC 客户端代理

**证据**: `services/zidl/BUILD.gn:33-55`

#### batterysrv_stub (ohos_source_set)

**文件路径**: `services/zidl/BUILD.gn:57-82`

**类型**: ohos_source_set

**sources**: `*_stub.cpp` (生成的 Stub 代码)

**deps**: `:batterysrv_interface`

**configs**:
- `${battery_utils}:utils_config`
- `:batterysrv_public_config`

**external_deps**:
- `c_utils:utils`
- `hilog:libhilog`
- `ipc:ipc_core`
- `ipc:ipc_single`

**说明**: IPC 服务端 Skeleton

**证据**: `services/zidl/BUILD.gn:57-82`

---

### Charger 层 Targets（可选）

#### charger (ohos_executable)

**文件路径**: `charger/BUILD.gn:...`

**类型**: ohos_executable

**install_enable**: true

**defines**:
- ENABLE_INIT_LOG
- DIFF_PATCH_SDK

**sources**:
- `animation_config.cpp`
- `battery_backlight.cpp`
- `battery_config.cpp`
- `battery_led.cpp`
- `battery_thread.cpp`
- `battery_vibrate.cpp`
- `charger.cpp`
- `charger_animation.cpp`
- `charger_graphic_engine.cpp`
- `charger_thread.cpp`
- `dev/drm_driver.cpp`
- `dev/fbdev_driver.cpp`
- `dev/graphic_dev.cpp`
- `power_supply_provider.cpp`

**configs**:
- `:batteryd_private_config`
- `${battery_utils}:coverage_flags`
- `./../utils:coverage_flags`
- `./../utils:utils_config`

**external_deps**:
- `cJSON:cjson`
- `c_utils:utils`
- `drivers_interface_battery:libbattery_proxy_2.0`
- `drivers_interface_input:libinput_proxy_1.0`
- `init:libbegetutil`
- `input:libmmi-client`
- `ipc:ipc_core`
- `libpng:libpng`

**条件 external_deps** (ENABLE_CHARGER 条件):
- `drivers_interface_display:libdisplay_composer_hdi_impl_1.3`
- `drivers_interface_display:libdisplay_composer_proxy_1.0`
- `drivers_interface_light:liblight_proxy_1.0`
- `graphic_surface:buffer_handle`
- `libdrm:libdrm`
- `ui_lite:libupdater_layout`
- `graphic_utils_lite:utils_lite`

**条件 defines**:
- `HAS_BATTERY_CONFIG_POLICY_PART` (has_battery_config_policy_part)
- `ENABLE_CHARGER` (battery_manager_feature_enable_charger)

**说明**: 关机充电可执行程序

**输出**: charger 可执行文件

**证据**: `charger/BUILD.gn:...`

#### charger_group (group)

**类型**: group

**条件**: `battery_manager_feature_enable_charger`

**deps**:
- `:charger`
- `:charger_animation`
- `:resources_service`

**说明**: 聚合所有充电相关 target

**证据**: `charger/BUILD.gn:...`

---

### Utils 层 Targets

#### battery_hookmgr (ohos_shared_library)

**文件路径**: `utils/hookmgr/BUILD.gn:...`

**类型**: ohos_shared_library

**sources**:
- `src/battery_hookmgr.cpp`

**configs**:
- `:private_config`
- `${battery_utils}:coverage_flags`
- `:public_config`

**public_configs**:
- `public_config`

**external_deps**:
- `c_utils:utils`
- `hilog:libhilog`
- `init:libbegetutil`

**sanitize**: cfi=true, cfi_cross_dso=true, debug=false

**branch_protector_ret**: pac_ret

**输出**: libbattery_hookmgr.z.so

**证据**: `utils/hookmgr/BUILD.gn:...`

---

### CJ FFI Targets

#### cj_battery_info_ffi (ohos_shared_library)

**文件路径**: `frameworks/cj/BUILD.gn:...`

**类型**: ohos_shared_library

**sources**:
- `src/battery_info_ffi.cpp`

**deps**:
- `${battery_inner_api}:batterysrv_client`

**configs**:
- `${battery_utils}:coverage_flags`

**public_configs**:
- `:ohbattery_info_config`

**external_deps**:
- `c_utils:utils`
- `hilog:libhilog`
- `ipc:ipc_core`
- `napi:ace_napi`
- `napi:cj_bind_ffi`
- `napi:cj_bind_native`

**innerapi_tags**: platformsdk

**sanitize**: integer_overflow=true, ubsan=true, boundary_sanitize=true, cfi=true, cfi_cross_dso=true

**输出**: libcj_battery_info_ffi.z.so

**证据**: `frameworks/cj/BUILD.gn:...`

---

### C API Targets

#### ohbattery_info (ohos_shared_library)

**文件路径**: `frameworks/capi/BUILD.gn:...`

**类型**: ohos_shared_library

**sources**:
- `./battery_info/ohbattery_info.cpp`

**public_configs**:
- `:ohbattery_info_config`

**deps**:
- `${battery_inner_api}:batterysrv_client`

**configs**:
- `${battery_utils}:coverage_flags`

**external_deps**:
- `c_utils:utils`
- `hilog:libhilog`
- `ipc:ipc_core`
- `samgr:samgr_proxy`

**output_extension**: so

**relative_install_dir**: ndk

**输出**: libohbattery_info.z.so

**证据**: `frameworks/capi/BUILD.gn:...`

---

## 编译产物清单

### Shared Libraries (共享库)

| 库名 | Target | 安装路径 | 说明 |
|------|--------|---------|------|
| libbatteryinfo.z.so | batteryinfo | /system/lib/module | N-API batteryInfo 模块 |
| libbattery.z.so | battery | /system/lib/module | N-API battery 模块（异步） |
| libcharger.z.so | charger | /system/lib/module | N-API charger 模块 |
| libbatteryservice.z.so | batteryservice | /system/lib | 电池服务 SA 库 |
| libbatterysrv_client.z.so | batterysrv_client | /system/platformsdk/lib | 电池服务客户端库 |
| libbattery_notification.z.so | battery_notification | /system/lib | 电池通知库 |
| libcj_battery_info_ffi.z.so | cj_battery_info_ffi | /system/lib | CJ FFI 库 |
| libbattery_hookmgr.z.so | battery_hookmgr | /system/lib | Hook 管理器库 |
| libohbattery_info.z.so | ohbattery_info | /system/lib/ndk | C API 库 |

**证据**: 各 target 的 `relative_install_dir` 配置

### Executables (可执行文件)

| 可执行文件 | Target | 安装路径 | 说明 |
|----------|--------|---------|------|
| charger | charger | /system/bin | 关机充电程序 |

**证据**: `charger/BUILD.gn:install_enable=true`

---

## 产物映射关系

### Target 到最终产物

```
N-API 模块
├── napi:battery
│   └──> libbatteryinfo.z.so (/system/lib/module/)
├── napi:batteryinfo
│   └──> libbattery.z.so (/system/lib/module/)
└── napi:charger
    └──> libcharger.z.so (/system/lib/module/)

C API
└── capi:ohbattery_info
    └──> libohbattery_info.z.so (/system/lib/ndk/)

CJ FFI
└── cj:cj_battery_info_ffi
    └──> libcj_battery_info_ffi.z.so (/system/lib/)

IPC 客户端
└── interfaces:inner_api:batterysrv_client
    └──> libbatterysrv_client.z.so (/system/platformsdk/lib/)

服务端
├── services:batteryservice
│   └──> libbatteryservice.z.so (/system/lib/)
├── services:battery_notification
│   └──> libbattery_notification.z.so (/system/lib/)
└── utils:hookmgr:battery_hookmgr
    └──> libbattery_hookmgr.z.so (/system/lib/)

ZIDL 生成
├── services/zidl:batterysrv_interface
│   └──> 生成 Proxy/Stub 代码
├── services/zidl:batterysrv_proxy
│   └──> 编译到 batteryservice
└── services/zidl:batterysrv_stub
    └──> 编译到 batteryservice

Charger（可选）
└── charger:charger
    └──> /system/bin/charger (可执行文件)
```

**证据**: 综合各 BUILD.gn 文件

---

## 安装路径说明

### 系统库路径

| 路径类型 | 路径 | 安装内容 |
|----------|------|----------|
| N-API 模块 | /system/lib/module/ | libbatteryinfo.z.so, libbattery.z.so, libcharger.z.so |
| SA 服务 | /system/lib/ | libbatteryservice.z.so, libbattery_notification.z.so |
| 系统库 | /system/lib/ | libbattery_hookmgr.z.so |
| C API | /system/lib/ndk/ | libohbattery_info.z.so |
| CJ FFI | /system/lib/ | libcj_battery_info_ffi.z.so |

### Platform SDK 路径

| 路径 | 安装内容 |
|------|----------|
| /system/platformsdk/lib/ | libbatterysrv_client.z.so |

**证据**: `interfaces/inner_api/BUILD.gn:48`

### 可执行文件路径

| 路径 | 安装内容 |
|------|----------|
| /system/bin/ | charger（可选） |

---

## 运行时加载关系

### SA 启动

```cpp
// SA 配置: sa_profile/3302.json
{
    "name": 3302,
    "libpath": "libbatteryservice.z.so",
    "run-on-create": true,
    "process": "powermgr"
}

// 系统启动时，powermgr 进程启动
// SA 框架检查 run-on-create，自动启动 BatteryService
// BatteryService 作为 libbatteryservice.z.so 被加载
```

**证据**: `sa_profile/3302.json:1-13`

### 应用层加载

```
JS 应用进程
    ↓
dlopen("libbatteryinfo.z.so") 或 "libbattery.z.so"
    ↓
调用 napi_module_register()
    ↓
模块初始化
```

**证据**: N-API 注册机制 `frameworks/napi/src/battery_info.cpp:619-622`

### IPC 连接

```cpp
JS 应用
    ↓
N-API: BatterySrvClient::GetInstance()
    ↓
连接 SAMGR
    ↓
查询 SA 3302 (powermgr 进程)
    ↓
获取 IBatterySrv 远程对象
    ↓
BatterySrvClient 存储代理，用于后续 IPC 调用
```

**证据**: `frameworks/native/src/battery_srv_client.cpp:35-65`

---

## 条件编译产物

### 特性开关影响

| 特性 | 关闭时 | 开启时 |
|------|----------|----------|
| battery_manager_feature_enable_charger | 不编译 charger 模块 | 编译 charger 可执行文件和依赖 |
| battery_manager_feature_enable_charging_sound | 不包含充电音效 | 添加 charging_sound target |
| battery_manager_feature_enable_wireless_charge | 不启用无线充电 | 启用 BATTERY_MANAGER_ENABLE_WIRELESS_CHARGE 宏 |
| battery_manager_feature_set_low_capacity_threshold | 不启用低电量关机 | 启用 BATTERY_MANAGER_SET_LOW_CAPACITY_THRESHOLD 宏 |
| battery_manager_feature_support_notification | 不启用通知功能 | 添加通知资源和依赖 |
| battery_manager_feature_support_notification_string | 不启用多语言 | 添加多语言资源文件 |

**证据**: `services/BUILD.gn` 中的条件 deps

### 部件检测影响

| 部件 | 缺失时 | 存在时 |
|------|----------|----------|
| has_sensors_miscdevice_part | 不添加 miscdevice 依赖 | 添加 light_interface_native 依赖 |
| has_drivers_interface_display_part | 不添加 display/light 驱动依赖 | 充电功能不编译 |
| has_drivers_interface_light_part | 不添加 light 驱动依赖 | 充电功能不编译 |
| has_battery_config_policy_part | 不使用 configpolicy_util | 使用默认配置值 |

**证据**: `batterymgr.gni:26-79`

---

## 相关跳转

- [系统架构](03_Architecture.md)
- [目录结构](02_Directory_Structure.md)

---

**返回**: [导航](SUMMARY.md)
