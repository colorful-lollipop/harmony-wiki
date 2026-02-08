# 构建配置

> UI Appearance GN 构建目标与编译产物说明

## 构建系统概述

UI Appearance 使用 OpenHarmony 的 **GN (Generate Ninja)** 构建系统。

### 构建命令

```bash
# 全量编译
./build.sh --product-name {product_name} --ccache --build-target ui_appearance_packages

# 单独编译 ui_appearance 部件
./build.sh --product-name {product_name} --ccache --build-target ui_appearance_packages
```

---

## 根构建入口

**文件**: `//foundation/arkui/ui_appearance:BUILD.gn`

```gn
import("//build/ohos.gni")

group("ui_appearance_packages") {
  deps = [
    "etc/para:ui_appearance.para",
    "etc/para:ui_appearance.para.dac",
    "interfaces/kits/napi:uiappearance",
    "sa_profile:arkui_ui_appearance_sa_profiles",
    "services:ui_appearance_service",
  ]
}
```

---

## 模块构建配置

### 1. N-API 模块

**路径**: `//foundation/arkui/ui_appearance/interfaces/kits/napi:BUILD.gn`

```gn
ohos_shared_library("uiappearance") {
  sources = [ "src/js_ui_appearance.cpp" ]
  
  include_dirs = [ "include/" ]
  
  external_deps = [
    "ipc:ipc_single",
    "napi:napi_native",
    "hilog:libhilog",
  ]
  
  subsystem_name = "arkui"
  part_name = "ui_appearance"
}
```

**输出产物**: `libuiappearance.z.so`

### 2. ETS 接口模块

**路径**: `//foundation/arkui/ui_appearance/interfaces/ets/ani:BUILD.gn`

```gn
ohos_builtin_ani("ui_appearance_ani_package") {
  sources = [ "src/ui_appearance.cpp" ]
  
  ets_sources = [ "ets/@ohos.uiAppearance.ets" ]
  
  subsystem_name = "arkui"
  part_name = "ui_appearance"
}
```

### 3. SA 服务模块

**路径**: `//foundation/arkui/ui_appearance/services:BUILD.gn`

#### IDL 接口生成

```gn
idl_gen_interface("ui_appearance_ability_interface") {
  sources = [ "IUiAppearanceAbility.idl" ]
  log_domainid = "0xD003900"
  log_tag = "UiAppearance"
  subsystem_name = "arkui"
  part_name = "ui_appearance"
}
```

#### SA Stub

```gn
ohos_source_set("ui_appearance_ability_stub") {
  sanitize = {
    cfi = true
    cfi_cross_dso = true
    debug = false
  }
  
  public_configs = [ ":ui_appearance_service_config" ]
  
  sources = filter_include(
    get_target_outputs(":ui_appearance_ability_interface"), 
    [ "*_stub.cpp" ])
  
  deps = [ ":ui_appearance_ability_interface" ]
  
  external_deps = [
    "c_utils:utils",
    "hilog:libhilog",
    "ipc:ipc_single",
    "samgr:samgr_proxy",
  ]
  
  subsystem_name = "arkui"
  part_name = "ui_appearance"
}
```

#### SA Service

```gn
ohos_shared_library("ui_appearance_service") {
  sanitize = {
    cfi = true           # 启用控制流完整性检测
    cfi_cross_dso = true # 跨 SO CFI 检查
    debug = false
  }

  sources = [
    "src/dark_mode_manager.cpp",
    "src/dark_mode_temp_state_manager.cpp",
    "src/screen_switch_operator_manager.cpp",
    "src/smart_gesture_manager.cpp",
    "src/ui_appearance_ability.cpp",
    "src/background_app_color_switch_settings.cpp",
    "utils/src/alarm_timer.cpp",
    "utils/src/alarm_timer_manager.cpp",
    "utils/src/json_utils.cpp",
    "utils/src/parameter_wrap.cpp",
    "utils/src/setting_data_manager.cpp",
    "utils/src/setting_data_observer.cpp",
  ]
  
  sources += filter_include(
    get_target_outputs(":ui_appearance_ability_interface"), 
    [ "*_stub.cpp" ])
  
  deps = [ ":ui_appearance_ability_interface" ]

  public_configs = [ ":ui_appearance_service_config" ]

  cflags_cc = [
    "-fvisibility=hidden",
    "-fvisibility-inlines-hidden",
    "-Oz",
    "-fdata-sections",
    "-ffunction-sections",
    "-fno-asynchronous-unwind-tables",
    "-fno-unwind-tables",
  ]
  
  ldflags = [ "-Wl,--gc-sections" ]

  include_dirs = [
    "include/",
    "utils/include/",
  ]

  external_deps = [
    "ability_base:configuration",
    "ability_runtime:app_manager",
    "ability_runtime:dataobs_manager",
    "ability_runtime:wantagent_innerkits",
    "access_token:libaccesstoken_sdk",
    "c_utils:utils",
    "common_event_service:cesfwk_core",
    "common_event_service:cesfwk_innerkits",
    "config_policy:configpolicy_util",
    "data_share:datashare_consumer",
    "hilog:libhilog",
    "init:libbegetutil",
    "ipc:ipc_single",
    "os_account:os_account_innerkits",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
    "time_service:time_client"
  ]
  
  subsystem_name = "arkui"
  part_name = "ui_appearance"
}
```

**输出产物**: `libui_appearance_service.z.so`

#### SA Client

```gn
ohos_shared_library("ui_appearance_client") {
  sources = [ "src/ui_appearance_ability_client.cpp" ]
  
  sources += filter_include(
    get_target_outputs(":ui_appearance_ability_interface"), 
    [ "*_proxy.cpp" ])
  
  deps = [ ":ui_appearance_ability_interface" ]
  
  public_configs = [ ":ui_appearance_service_config" ]
  
  include_dirs = [ "include/" ]

  external_deps = [
    "c_utils:utils",
    "hicollie:libhicollie",
    "hilog:libhilog",
    "ipc:ipc_single",
    "safwk:system_ability_fwk",
    "samgr:samgr_proxy",
  ]

  cflags_cc = [
    "-fvisibility=hidden",
    "-fvisibility-inlines-hidden",
    "-Oz",
    "-fdata-sections",
    "-ffunction-sections",
    "-fno-asynchronous-unwind-tables",
    "-fno-unwind-tables",
  ]
  
  ldflags = [ "-Wl,--gc-sections" ]

  subsystem_name = "arkui"
  innerapi_tags = [ "platformsdk" ]
  part_name = "ui_appearance"
}
```

### 4. 配置文件

**路径**: `//foundation/arkui/ui_appearance/etc/para:BUILD.gn`

```gn
ohos_prebuilt_etc("ui_appearance.para") {
  source = "ui_appearance.para"
  subsystem_name = "arkui"
  part_name = "ui_appearance"
  module_install_dir = "etc/param"
}

ohos_prebuilt_etc("ui_appearance.para.dac") {
  source = "ui_appearance.para.dac"
  subsystem_name = "arkui"
  part_name = "ui_appearance"
  module_install_dir = "etc/param"
}
```

### 5. SA Profile

**路径**: `//foundation/arkui/ui_appearance/sa_profile:BUILD.gn`

```gn
ohos_sa_profile("arkui_ui_appearance_sa_profiles") {
  sources = [ "7002.json" ]
  subsystem_name = "arkui"
  part_name = "ui_appearance"
}
```

---

## 产物清单

### 编译产物

| 产物 | 路径 | 说明 |
|------|------|------|
| libuiappearance.z.so | `out/{product}/arkui/ui_appearance/` | N-API 库 |
| libui_appearance_service.z.so | `out/{product}/arkui/ui_appearance/` | SA 服务库 |
| libui_appearance_client.z.so | `out/{product}/arkui/ui_appearance/` | SA 客户端库 |
| ui_appearance.para | `out/{product}/arkui/ui_appearance/` | 参数配置 |
| ui_appearance.para.dac | `out/{product}/arkui/ui_appearance/` | DAC 配置 |
| arkui_ui_appearance_sa_profiles | `out/{product}/arkui/ui_appearance/` | SA 配置 |

### 安装产物

| 产物 | 目标路径 | 说明 |
|------|----------|------|
| libuiappearance.z.so | `system/lib64/` | N-API 库 |
| libui_appearance_service.z.so | `system/lib64/` | SA 服务库 |
| libui_appearance_client.z.so | `system/lib64/` | SA 客户端库 |
| ui_appearance.para | `system/etc/param/` | 参数配置 |
| ui_appearance.para.dac | `system/etc/param/` | DAC 配置 |

---

## 依赖关系

### 内部依赖

```
ui_appearance_packages (group)
    ├── etc/para:ui_appearance.para
    ├── etc/para:ui_appearance.para.dac
    ├── interfaces/kits/napi:uiappearance
    │   └── depends on: ipc, napi, hilog
    ├── sa_profile:arkui_ui_appearance_sa_profiles
    └── services:ui_appearance_service
        ├── ui_appearance_ability_stub
        │   └── depends on: ipc, samgr, c_utils, hilog
        ├── ui_appearance_ability_interface (IDL)
        └── depends on:
            ├── ability_base (configuration)
            ├── ability_runtime (app_manager, dataobs_manager)
            ├── access_token
            ├── c_utils
            ├── common_event_service
            ├── config_policy
            ├── data_share
            ├── hilog
            ├── init
            ├── ipc
            ├── os_account
            ├── safwk
            ├── samgr
            └── time_service
```

### 外部依赖 (bundle.json)

```json
{
  "deps": {
    "components": [
      "ability_runtime",
      "ability_base",
      "access_token",
      "c_utils",
      "config_policy",
      "data_share",
      "hicollie",
      "hilog",
      "init",
      "ipc",
      "napi",
      "safwk",
      "samgr",
      "time_service",
      "os_account",
      "common_event_service",
      "runtime_core"
    ],
    "third_party": []
  }
}
```

---

## 组件配置

**文件**: `//foundation/arkui/ui_appearance/bundle.json`

```json
{
  "name": "@ohos/ui_appearance",
  "description": "Provide ui_appearance management.",
  "version": "3.2",
  "component": {
    "name": "ui_appearance",
    "subsystem": "arkui",
    "syscap": [
      "SystemCapability.ArkUI.UiAppearance"
    ],
    "features": [],
    "adapted_system_type": [
      "standard"
    ],
    "rom": "300KB",
    "ram": "1024KB",
    "build": {
      "sub_component": [
        "//foundation/arkui/ui_appearance:ui_appearance_packages",
        "//foundation/arkui/ui_appearance/interfaces/ets/ani:ui_appearance_ani_package"
      ],
      "inner_kits": [],
      "test": []
    }
  }
}
```

---

## 资源占用

| 资源类型 | 大小 | 说明 |
|----------|------|------|
| ROM | 300KB | 编译后代码大小 |
| RAM | 1024KB | 运行时内存占用 |

---

## SA 配置

**文件**: `//foundation/arkui/ui_appearance/sa_profile/7002.json`

```json
{
    "process": "ui_service",
    "systemability": [
        {
            "name": 7002,
            "libpath": "libui_appearance_service.z.so",
            "run-on-create": true,
            "distributed": false,
            "dump_level": 1,
            "min_hdi_proxy_version": []
        }
    ]
}
```

**SA 属性**:
- **SA ID**: 7002
- **进程名**: ui_service
- **库路径**: libui_appearance_service.z.so
- **启动方式**: run-on-create = true (系统启动时创建)
- **分布式**: false
- **dump_level**: 1

---

*相关内容: [N-API 接口](03_NAPI.md) | [安全评审](06_Security.md)*
